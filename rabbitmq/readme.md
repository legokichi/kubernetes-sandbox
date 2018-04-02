# ありがちな構成の kubernetes objects

* nfs: ファイルサーバ
* node: Webサーバ
* mongodb: 顧客情報など
* redis: タスクキューと終了通知
* somejob: バッチジョブサービス

```
ConfigMap Secret    ConfigMap   Secret
  |         |         |           |
DaemonSet(fluentd)  DaemonSet(prometheus)   Deployment(minion)
                                                 |
PersistentVolume(NFS)                       Service(minion)
        |
PersistentVolumeClaim -----------+---------------------------------------+
        |                        |                                       |
Secret  | ConfigMap              |              ConfigMap         Secret | ConfigMap
   |    |    |                   |                  |                 |  |    |
StatefulSet(MongoDB)      Secret | ConfigMap    StatefulSet(redis)   Deployment(somejob)
        |                    |   |  |               |                   |
  Service(MongoDB) -------> Deployment(node) <-- Service(redis) ---> Service(somejob)
                                 |
                                 |             Secret ConfigMap, 
                                 |               |       |
                             Service(node) ---> Deployment(nginx)
                                                    |
                                                Service(nginx) -> Ingress(nginx)
```

* https://kubernetes.io/docs/concepts/services-networking/service/
* Service - namespaceクラスタ内ドメイン名
  * type: ClusterIP - クラスタ内部からのみ到達可能な IP として公開 
  * type: NodePort - 外部から <NodeIP>:<NodePort> として到達可能なように公開。裏で ClusterIP Service を作ってる
  * type: LoadBalancer - クラウドが用意したロードバランサ。クラスタ外部へ公開するときに使う
* Ingress - 実質 Service: type: LoadBalancer 。細かな設定ができる


## ありがちな構成の job queue

* https://qiita.com/MahoTakara/items/e0aa723b02d04e1accd2#jobのコンテナの作成
* https://kubernetes.io/docs/tasks/job/coarse-parallel-processing-work-queue/

```
Deployment(RabbitMQ)
    |
Service(RabbitMQ)  ---> Config
                         |
                        Job(gpujob)
```

```sh
sudo apt-get install -y amqp-tools 
docker run -p 15672:5672 rabbitmq:3
```

```sh
export BROKER_URL=amqp://guest:guest@localhost:15672
export QUEUE=openpose

: "create queue"
amqp-declare-queue --url=$BROKER_URL -q $QUEUE -d

: "enqueue"
find /path/to/ -name "*.mp4" -type f -print0 | xargs --null -I {} -- amqp-publish --url=$BROKER_URL -r $QUEUE -p -b {} 
# or
amqp-publish --url=$BROKER_URL -r $QUEUE -p -b /path/to/movie.mp4

: "process"
amqp-consume --url=$BROKER_URL -q $QUEUE -- xargs -I {} -- env NV_GPU=0 bash worker.bash {}
# or 
amqp-consume --url=$BROKER_URL -q $QUEUE cat 
```

```sh
#!/bin/bash
set -euvx
: "usage: cli"
: '  env NV_GPU=0 bash worker.bash /path/to/0.mp4'
: '  find /path/to/ -name "*.mp4" -type f -print0 | xargs -I {} -- env NV_GPU=0 bash worker.bash {}'

: "usage: rabbitmq"
: "  sudo apt-get install -y amqp-tools"
: "  docker run -p 15672:5672 rabbitmq:3"
: "  export BROKER_URL=amqp://guest:guest@localhost:15672"
: "  export QUEUE=openpose"
: "  amqp-declare-queue --url=$BROKER_URL -q $QUEUE -d"
: '  find /path/to/ -name "*.mp4" -type f -print0 | xargs -I {} -- amqp-publish --url=$BROKER_URL -r $QUEUE -p -b {}'
: "  amqp-consume --url=$BROKER_URL -q $QUEUE -- xargs -I {} -- env NV_GPU=0 bash worker.bash {}"

while [ $# -gt 0 ];
do
  date
  SRC="$1"
  BASE=$(basename -s .mp4 "$SRC")
  DST=$(dirname $(realpath "$SRC"))
  echo "$SRC" "$BASE" "$DST"
  # ffmpeg -y -i "$SRC" "$DST/$BASE".avi
  time sudo env NV_GPU=$NV_GPU nvidia-docker run \
    -e NVIDIA_VISIBLE_DEVICES="$NV_GPU" \
    --rm \
    -u $(id -u $(whoami)) \
    -v "$DST:$DST" \
    --workdir /opt/openpose-docker \
    openpose:cli-1.2.1 \
      /opt/openpose-docker/build/examples/openpose/openpose.bin \
        --display 0 \
        --num_gpu 1 \
        --video "$SRC" \
        --write_video "$DST/$BASE".openpose.avi \
        --write_json "$DST/$BASE".openpose.log &&:
  if [ $? -eq 0 ]; then
    time ffmpeg -y \
      -i "$DST/$BASE".openpose.avi \
      -c:v libx264 \
        -profile:v baseline \
        -level 3.1 \
        -pix_fmt yuv420p \
        -threads 4 \
      -f mp4 "$DST/$BASE".openpose.mp4
    rm -f "$DST/$BASE".openpose.avi
    sudo chown $(id -u $(whoami)):$(id -g $(whoami)) "$DST/$BASE".openpose.mp4
    sudo chown -R $(id -u $(whoami)):$(id -g $(whoami)) "$DST/$BASE".openpose.log
    ls -la "$DST"
    echo "openpose OK $SRC"
  else
    amqp-publish --url=$BROKER_URL -r $QUEUE -p -b "$SRC"
    echo "openpose NG $SRC"
  fi
  #rm -f "$DST/$BASE".avi
  shift
done
```