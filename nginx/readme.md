
```sh
kubectl create -f nginx.yml
kubectl get pod
export POD_NAME=nginx-pod
kubectl describe pods $POD_NAME
kubectl logs $POD_NAME
kubectl exec -it $POD_NAME /bin/bash
kubectl exec $POD_NAME env
kubectl exec $POD_NAME -- ls -la /
kubectl delete pod $POD_NAME
```
