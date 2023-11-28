
Metallb
https://metallb.universe.tf/

0. Если ставим через kubespray то посмотреть в inventory/**vdsina**/group_vars/k8s_cluster/k8s-cluster.yml
```yaml
# Kube-proxy proxyMode configuration.
# Can be ipvs, iptables
kube_proxy_mode: ipvs

# configure arp_ignore and arp_announce to avoid answering ARP queries from kube-ipvs0 interface
# must be set to true for MetalLB, kube-vip(ARP enabled) to work
kube_proxy_strict_arp: true
```
1. Если уже готовый кластер посмотреть `kubectl edit configmap -n kube-system kube-proxy`
Убедится, что KubeProxy запущен с параметром:

```yaml
ipvs:
  strictARP: true
```

1. При использовании meetallb > v0.13 надо вместо ConfigMap использовать IPAddressPool и L2Advertisement
2. При применении может случится так, что будет ошибка с webhook. Лечится удалением ` kubectl delete ValidatingWebhookConfiguration metallb-webhook-configuration`

## Установка Helm'ом
helm upgrade --install metallb metallb/metallb --namespace metallb-system --create-namespace
Лучше удалить и заново создать
kubectl create secret generic -n metallb-system metallb-memberlist --from-literal=secretkey="$(openssl rand -base64 128)"

MetalLB never allocates node IPs. If you try to do that, network routing
within kubernetes will most likely break - or at least I can't guarantee
anything about it working.

MetalLB wants a separate set of virtual IPs to allocate, distinct from node
IPs. This allows the IPs to float between nodes on failover



kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/metallb.yaml

Только при первой установке создаём сикрет:

kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"

Применяем базовую конфигурацию layer2.

kubectl apply -f metallb/00-mlb.yaml

Добавляем сервис для ingres-controller типа LoadBalancer

kubectl apply -f metallb/01-lb-ingress-controller-svc.yaml