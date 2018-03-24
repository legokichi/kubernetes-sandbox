# ありがちな構成の kubernetes objects
* nfs: ファイルサーバ
* node: Webサーバ
* mongodb: 顧客情報など
* redis: タスクキューと終了通知
* somejob: バッチジョブサービス

```
ConfigMap Secret
  |         |
DaemonSet(fluentd)

PersistentVolume(NFS)
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
