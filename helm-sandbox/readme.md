* 今日から始める人のためのKubernetes on AWSベストプラクティス - https://qiita.com/mumoshu/items/8ca3a7c1c8f48e02a658

## minikube

```sh
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

minikube start
eval $(minikube docker-env)
minikube dashboard
minikube stop
```

## kubectl

```sh
sudo snap install kubectl --classic
```

## kubectx + kubens
* https://github.com/ahmetb/kubectx

### kubectx
クラスタ切り替え

```console
$ kubectx
minikube
$ kubectx minikube
Switched to context "minikube".
```

### kubens
クラスタ内の名前空間切り替え

```console
$ kubens
default
kube-public
kube-system
$ kubens kube-system
Context "minikube" modified.
Active namespace is "kube-system".
$ kubectl get pods
NAME                                    READY     STATUS    RESTARTS   AGE
etcd-minikube                           1/1       Running   0          19m
kube-addon-manager-minikube             1/1       Running   3          2h
kube-apiserver-minikube                 1/1       Running   0          19m
kube-controller-manager-minikube        1/1       Running   0          19m
kube-dns-86f4d74b45-spkgf               3/3       Running   8          2h
kube-proxy-l99zw                        1/1       Running   0          18m
kube-scheduler-minikube                 1/1       Running   2          2h
kubernetes-dashboard-5498ccf677-np9hd   1/1       Running   6          2h
storage-provisioner                     1/1       Running   6          2h
tiller-deploy-f9b8476d-99wl2            1/1       Running   1          2h
```

### kustomize
* https://github.com/kubernetes-sigs/kustomize

複数の kubernetes ファイルから kubernetes を生成するコンパイラっぽい

## helm
helm は https://github.com/kubernetes/helm/releases からダウンロード

* helmとは & helmの使い方 - https://qiita.com/sheepland/items/75b647b71c34c7d38804
* helm + minikube + prasma - http://blog.soushi.me/entry/2018/01/30/071628
* helmを使ってKubernetesを楽にする - https://qiita.com/Hiroyuki_OSAKI/items/8965ceb6c90bae3bea76
* Helmの概要とChart(チャート)の作り方 - https://qiita.com/thinksphere/items/5f3e918015cf4e63a0bc

```sh
helm init
helm create myproduct
```

### helmfile
* https://qiita.com/micolash/items/9e4e7de201439dc00a4e#リリースは小さくする
* https://github.com/roboll/helmfile

これつかうよりかは skaffold + helm のほうがよさげ

## skaffold

```sh
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64 && chmod +x skaffold && sudo mv skaffold /usr/local/bin
```

* SkaffoldとDraftを比べてみた - https://qiita.com/watawuwu/items/7f5e287c91e5eab30419

### keel
* https://github.com/keel-hq/keel

### azure draft
* https://github.com/Azure/draft

## ingress + nginx

* https://qiita.com/MahoTakara/items/cdfa379f2280a58fdd6d
* ingress の役割
* virtual hosting の解決
* ssl/tls
* ロードバランス
* nginx の抽象化概念

