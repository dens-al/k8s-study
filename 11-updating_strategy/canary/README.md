microk8s:

microk8s enable metallb 
microk8s enable community
microk8s enable istio

kiali
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.18/samples/addons/kiali.yaml
prom
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.18/samples/addons/prometheus.yaml
jager
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.18/samples/addons/jaeger.yaml
grafana
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.18/samples/addons/grafana.yaml

kubectl create namespace canary
kubectl label namespace canary istio-injection=enabled

kubectl apply -f 11-updating_strategy/canary/manifests/gateway.yaml
kubectl apply -f 11-updating_strategy/canary/manifests/destination_rule.yam
kubectl apply -f 11-updating_strategy/canary/manifests/20-deployment-canary.yaml
