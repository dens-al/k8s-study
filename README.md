<details>
  <summary>Lesson 1. Basic in k8s</summary>

- Pods

```shell
kubectl apply -f 01-pods/01_pod_nginx.yaml
```

```shell
kubectl get pods
```

```shell
kubectl get pods -o wide
```

```shell
kubectl delete -f 01-pods/01_pod_nginx.yaml
kubectl apply -f 01-pods/01_pod_nginx.yaml
sleep 5
kubectl get pods -o wide
```
```shell
kubectl run my-curl-pod --image=curlimages/curl -it --rm -- sh 
```
- Labels

```shell
kubectl get pods --show-labels
```

```shell
kubectl label pods pod-nginx rock=metallica
```

```shell
kubectl get pods --show-labels
```

```shell
kubectl label pods pod-nginx rock=ac/dc
```

```shell
kubectl label pods pod-nginx rock=pink floyd
```

```shell
kubectl label pods pod-nginx rock=slayer
```

```shell
kubectl label pods pod-nginx alternative=korn
```

```shell
kubectl get pods --show-labels
```

```shell
kubectl get pods -l alternative=korn
```

- services and endpoints

```shell
kubectl apply -f 01-pods/02_pod_svc_nginx.yaml
```

```shell
kubectl get pods --show-labels
```

```shell
kubectl run my-curl-pod --image=curlimages/curl -it --rm -- sh 
curl svc-nginx2
```

```shell
kubectl get svc -o wide
```

```shell
kubectl get ep
```

```shell
kubectl describe svc svc-nginx2
```

```shell
kubectl apply -f 01-pods/03_pod_multitool.yaml 
```

```shell
kubectl exec pod-multitool -it -- bash
```

```shell
kubectl port-forward
```

</details>

<details>
  <summary>Lesson 2. Init containers, deployments, daemonsets</summary>

```shell
kubectl create ns lesson2
```

```shell
kubectl -n lesson2 get pod -w
```

```shell
  kubectl -n lesson2 logs pod-init -f
  ```

```shell
kubectl -n lesson2 logs pod-init -c delay
```

```shell
kubectl -n lesson2 describe pod init-pod
```

```shell
kubectl apply -f 01-pods/11_pod_init.yaml
```

```shell
kubectl apply -f 01-pods/12_pod_startup.yaml
```

```shell
kubectl apply -f 01-pods/13_pod_liveness.yaml
```

```shell
kubectl apply -f 01-pods/14_pod_readyness.yaml
```

```shell
kubectl apply -f 02-deployments/11_dpl_svc_nginx.yaml
```

```shell
kubectl -n lesson2 get replicaset
```

```shell
kubectl -n lesson2 delete pod dpl-nginx-65848665bd-97v6r
```

```shell
kubectl apply -f 03-daemonsets/11_dms.yaml
```

```shell
kubectl get pods -n lesson2 -o wide
```

</details>

### Lesson 3. Services

kubectl create ns lesson3

kubectl describe 
