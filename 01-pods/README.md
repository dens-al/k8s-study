
kubectl run curlpod --image=curlimages/curl -i --tty --rm -- sh

## Init containers
kubectl get pod -w 
kubectl logs init-pod -f
kubectl logs init-pod -c delay
kubectl describe pod init-pod
