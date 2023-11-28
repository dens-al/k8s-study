### Lesson 1. Basic in k8s

kubectl apply -f 01-pods/01_pod_nginx.yaml
kubectl get pods --show-labels
kubectl label pods pod-nginx rock=metallica
kubectl get pods --show-labels
kubectl label pods pod-nginx rock=ac/dc 
kubectl label pods pod-nginx rock=pinkfloyd
kubectl label pods pod-nginx rock-

kubectl apply -f 01-pods/02_pod_svc_nginx.yaml
kubectl run curl svc-nginx2 --image=curlimages/curl -i --tty --rm -- sh
kubectl get svc -o wide 
kubectl get ep
kubectl describe svc svc-nginx2

kubectl apply -f 01-pods/03_pod_multitool.yaml 
kubectl exec -it pod-multitool -- bash

### Lesson 2. Init containers, deployments, daemonsets
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

### Lesson 3. Services
kubectl create ns lesson3

kubectl describe 
