kubernetes の知見とか

* kubectlチートシート - https://qiita.com/mumoshu/items/19392308cdadf8667fdd
* Kubernetes & GKE チートシート - https://qiita.com/nirasan/items/d61be5cd4e286eb092cc
* kubectlコマンドの使い方(1.2) - https://qiita.com/hana_shin/items/ef1a20239001ac83a78d#32-kubectl-runkubectl-getkubectl-delete

```bash
minikube start
eval $(minikube docker-env)
minikube dashboard
minikube stop
```

```bash
kubectl cluster-info
kubectl api-versions
kubectl get componentstatus
kubectl config view
```

```bash
kubectx
kubectl config current-context

kubectx minikube
kubectl config minikube
```

```bash
kubens
kubens kube-system
kubens default
```

```
kubectl create namespace name1
kubectl delete namespace emojivoto
```

```
kubectl explain node
kubectl explain pod
kubectl explain deployment
kubectl explain rc
```

```bash
kubectl run <DEPLOYMENT NAME/POD NAME PREFIX> --image=<IMAGE>:<TAG> <CMD>
kubectl run mypod --tty -i --image=alpine:edge sh
```


```bash
kubectl create -f nginx.yml
kubectl replace -f nginx.yml
kubectl apply -f ./settings
curl https://foo/emojivoto.yml | kubectl apply -f -
kubectl rolling-update nginx --image=nginx:1.9.1
kubectl rollout history deployment nginx
kubectl rollout undo deployment nginx --to-revision=1
kubectl scale deployment nginx --replicas=2
```

```bash
kubectl delete pod $POD_NAME
kubectl delete service $SERVICE_NAME
for DEPLOYMENT in $(kubectl get deployments -o json | jq -r '.items[].metadata.name'); do kubectl delete deployment  $DEPLOYMENT; done
kubectl delete deployment $DEPLOYMENT
```

```bash
kubectl get pods
kubectl get pod --show-labels
kubectl get pods --all-namespaces
kubectl get pods --all-namespaces -o wide
kubectl get deployments
kubectl get service
kubectl get svc,pod,rc
```

```bash
kubectl describe nodes kubernetes-node-emt8.c.myproject.internal
kubectl describe pods/nginx
kubectl describe -f pod.json
kubectl describe pods
```

```bash
kubectl exec -it ${POD_NAME} sh
kubectl exec -it $POD_NAME /bin/bash
kubectl exec $POD_NAME env
kubectl exec $POD_NAME -- ls -la /
```

```bash
kubectl logs $POD_NAME
kubectl logs -f ${POD_NAME}
kubectl logs -p ${POD_NAME}
```


