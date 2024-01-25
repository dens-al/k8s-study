<details>
  <summary>Lesson 1. Basic in k8s</summary>
  
- Labels

```shell
kubectl apply -f 01-pods/01_pod_nginx.yaml
```

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
kubectl run curl svc-nginx2 --image=curlimages/curl -i --tty --rm -- sh
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
kubectl exec -it pod-multitool -- bash
```

</details>

<details>
  <summary>Lesson 2. Init containers, deployments, daemonsets</summary>
  
  kubectl create ns lesson2
  kubectl -n lesson2 get pod -w
  kubectl -n lesson2 logs pod-init -f
  kubectl -n lesson2 logs pod-init -c delay
  kubectl -n lesson2 describe pod init-pod
  kubectl apply -f 01-pods/11_pod_init.yaml
  kubectl apply -f 01-pods/12_pod_startup.yaml
  kubectl apply -f 01-pods/13_pod_liveness.yaml
  kubectl apply -f 01-pods/14_pod_readyness.yaml
  
  kubectl apply -f 02-deployments/11_dpl_svc_nginx.yaml
  kubectl -n lesson2 get replicaset
  kubectl -n lesson2 delete pod dpl-nginx-65848665bd-97v6r
  
  kubectl apply -f 03-daemonsets/11_dms.yaml
  kubectl get pods -n lesson2 -o wide
  
</details>

### Lesson 3. Services

kubectl create ns lesson3

kubectl describe 
