* Kubernetes: アプリケーションのデバッグ方法 - https://qiita.com/tkusumi/items/a62c209972bd0d4913fc
* 逆引き Kubernetes オブジェクト - https://qiita.com/iijimakazuyuki/items/465e7d697c3c02d317e6

### install k8s tools

```sh
sudo apt-get update
sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update -y
sudo apt-get install -y kubelet kubeadm kubectl
```

### install k8s

```sh
sudo kubeadm reset
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### install SDN

```sh
kubectl apply -f https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml
kubectl get pods --all-namespaces 
```

### enable master isolation

```sh
kubectl taint nodes --all node-role.kubernetes.io/master-
```


### install k8s-dashboard

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
#kubectl proxy --port=8001
export DASHBOARD_POD=$(kubectl get pods --namespace=kube-system |grep dashboard| awk '{print $2}')
kubectl port-forward --namespace=kube-system $DASHBOARD_POD 18001:8001
open https://localhost:8001/ui
```

### enable device plugin

```sh
sudo bash -c 'echo "Environment=\"KUBELET_EXTRA_ARGS=--feature-gates=DevicePlugins=true\"" >> /etc/systemd/system/kubelet.service.d/10-kubeadm.conf'
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v1.9/nvidia-device-plugin.yml
```

### fix apiserver ip address

```sh
sudo bash -c 'echo "Environment=\"KUBELET_EXTRA_ARGS=--node-ip=192.168.43.171\"" >> /etc/systemd/system/kubelet.service.d/10-kubeadm.conf'
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

* https://github.com/kubernetes/kubeadm/issues/203#issuecomment-359789817

### install helm

* https://github.com/kubernetes/helm
* Kubernetes: パッケージマネージャHelm - https://qiita.com/tkusumi/items/12857780d8c8463f9b9c

```sh
helm init
helm search prometheus
kubectl create clusterrolebinding permissive-binding --clusterrole=cluster-admin --user=admin --user=kubelet --group=system:serviceaccounts
helm install stable/prometheus-node-exporter
```

* https://stackoverflow.com/questions/43499971/helm-error-no-available-release-name-found
* https://github.com/kubernetes/helm/issues/2224
* https://github.com/kubernetes/helm/issues/3371

