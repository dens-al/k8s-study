helm repo list
helm search repo nginx
helm install web bitnami/nginx
helm list
helm install web bitnami/nginx --dry-run
helm pull bitnami/nginx
## Update rollback
helm update web
helm rollback web
helm history web
## Modify
helm show values bitnami/nginx
helm install web bitnami/nginx --set image.tag=1.20 --dry-run 
helm install web bitnami/nginx -f ./newvalues.yaml --dry-run
## Template
helm create first
helm template first
## Subchart
helm fetch bitnami/wordpress
