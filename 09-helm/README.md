### Helm as Packet Manager
helm repo list
helm search repo nginx

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install web bitnami/nginx
helm list -A
helm install web bitnami/nginx --dry-run
helm status web
helm install web bitnami/nginx
helm install web2 bitnami/nginx --namespace netology
helm install web bitnami/nginx --namespace web --create-namespace
helm pull bitnami/nginx

## Update rollback
kubectl get pods -n web -o yaml | grep image
helm upgrade --install web bitnami/nginx --namespace web --set image.tag=1.20 --dry-run
helm upgrade --install web bitnami/nginx --namespace web --set image.tag=1.20
helm list -n web 
helm history web -n web
kubectl get pods -n web -o yaml | grep image
helm upgrade --install web bitnami/nginx --namespace web --set image.tag=11.20
kubectl get pods -n web
helm rollback -n web web 4
helm history web -n web
helm upgrade --install --atomic --timeout 20s web bitnami/nginx --namespace web --set image.tag=11.20

## Modify
helm show values bitnami/nginx
helm install web bitnami/nginx --set image.tag=1.20 --dry-run 
helm install web bitnami/nginx -f ./newvalues.yaml --dry-run
helm pull bitnami/nginx

## Template and package
helm create app
helm template app
helm package 09-helm/app

## Subchart
helm fetch bitnami/wordpress
