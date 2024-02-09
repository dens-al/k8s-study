<details>
  <summary>Lesson 1. Basic in k8s</summary>

### Pods

```shell
kubectl apply -f 01-pods/01_pod_nginx.yaml
```

- посмотреть все поды

```shell
kubectl get pods
```

- посмотреть подробнее про все поды в том числе IP-адреса

```shell
kubectl get pods -o wide
```

- IP адреса меняются после перезапуска пода

```shell
kubectl delete -f 01-pods/01_pod_nginx.yaml
kubectl apply -f 01-pods/01_pod_nginx.yaml
sleep 5
kubectl get pods -o wide
```

- запуск пода императивно

```shell
kubectl run my-curl-pod --image=curlimages/curl -it --rm -- sh 
```

### Labels

- просмотр подов с метками

```shell
kubectl get pods --show-labels
```

- добавление метки

```shell
kubectl label pods pod-nginx rock=metallica
kubectl get pods --show-labels
```

- не все знаки можно использовать в метках

```shell
kubectl label pods pod-nginx rock=ac/dc
kubectl label pods pod-nginx rock=pink floyd
```

- ключ должен быть уникальный

```shell
kubectl label pods pod-nginx rock=slayer
kubectl label pods pod-nginx alternative=korn
```

```shell
kubectl get pods --show-labels
kubectl get pods -L alternative,rock
```

- фильтрация по метке

```shell
kubectl get pods -l alternative=korn
kubectl get pods -l alternative!=korn
```

- фильтрация по набору

```shell
kubectl get pods -l alternative,rock
```

- можно метить ноды и использовать в NodeSelecor

```shell
kubectl get pods 
kubectl label nodes microk8s-01 gpu=true
kubectl get pods 
```

- удалить метку

```shell
kubectl label nodes microk8s-01 gpu-
```

- добавление аннотации

```shell
kubectl annotate pod pod-nginx created-by="denis"
kubectl describe pod pod-nginx
kubectl describe pod pod-nginx-node-selector
```

### Services and endpoints

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

### Init Containers

- создадим namespace, в котором будем устанавливать ресурсы

```shell
kubectl create ns lesson2
```

- запустим просмотр состояния подов в интерактивном режиме (watch)

```shell
kubectl -n lesson2 get pod -w
```

- перейдем на другую консоль и запустим инит контейнера

```shell
kubectl apply -f 01-pods/11_pod_init.yaml
```

- после запуска можно посмотреть логи и увидеть как запускался инит контейнер

```shell
kubectl -n lesson2 describe pod pod-init
```

### Probes

```shell
kubectl apply -f 01-pods/12_pod_startup.yaml
```

```shell
kubectl apply -f 01-pods/13_pod_liveness.yaml
```

```shell
kubectl apply -f 01-pods/14_pod_readyness.yaml
```

- почему не запустился под? Потому что readinessProbe проверяет другой порт. Endpoints не появился, хотя сервис есть.

### Deployments

```shell
kubectl apply -f 02-deployments/11_dpl_svc_nginx.yaml
kubectl get pods -n lesson2
```

- названия подов состоят из имени деплоймента, имени репликисет и собственного хвоста

```shell
kubectl -n lesson2 get replicaset
```

- попробуем удалить один произвольный под из деплоймента и увидем, что деплоймент запустит НОВЫЙ под с другим именем.

```shell
kubectl -n lesson2 delete pod dpl-nginx-65848665bd-97v6r
kubectl get pods -n lesson2 -w
```

- демонсет

```shell
kubectl apply -f 03-daemonsets/11_dms.yaml
```

</details>


<details>
  <summary>Lesson 3. Services</summary>

### Cluster IP

- создадим namespace, в котором будем устанавливать ресурсы

```shell
kubectl create ns lesson3
```

- установим деплоймент с мултитулом и сервис. Портам можно давать имена в конфигурации, чтобы в дальнейшем обращаться по
  имени

```shell
kubectl apply -f 04-services/21_svc_clusterip_multitool.yaml
kubectl describe svc -n lesson3 svc-multitool-clusterip
```

создадим pod с curl и изучим адреса и доменные имена сервиса

```shell
kubectl run -n lesson3 my-curl-pod --image=curlimages/curl -it --rm -- sh 
curl svc-multitool-clusterip
curl svc-multitool-clusterip.lesson3.svc.cluster.local
nslookup svc-multitool-clusterip.lesson3.svc.cluster.local
```

- убедимся, что можно из **другого** неймспейса достучаться по днс-имени

```shell
kubectl run my-curl-pod --image=curlimages/curl -it --rm -- sh 
curl svc-multitool-clusterip
curl svc-multitool-clusterip.lesson3.svc.cluster.local
nslookup svc-multitool-clusterip.lesson3.svc.cluster.local
```

- рассмотрим сервис с несколькими портами и pod с несколькими контейнерами

```shell
kubectl apply -f 04-services/22_svc_multi_dpl_multitool.yaml
kubectl describe svc -n lesson3 svc-multitool-clusterip-multiport
kubectl get ep -n lesson3
```

- убедимся, что можно из достучаться через разные порты одного сервиса на разные контейнеры

```shell
kubectl run -n lesson3 my-curl-pod --image=curlimages/curl -it --rm -- sh 
curl svc-multitool-clusterip-multiport:1601
curl svc-multitool-clusterip-multiport:1602
curl -k https://svc-multitool-clusterip-multiport:1603
```

### NodePort

- применим манифест с сервисом типа NodePort

```shell
kubectl apply -f 04-services/23_svc_nodeport_multitool.yaml
kubectl get svc -n lesson3 -o wide
kubectl get ep -n lesson3
kubectl get nodes -o wide
```

- убедимся, что можно подключиться к **любой** ноде по порту 30080, даже если pod запущен на другой ноде

```shell
curl 158.160.117.28:30080
```

- при этом сервис по прежнему доступен внутри кластера

```shell
kubectl run -n lesson3 my-curl-pod --image=curlimages/curl -it --rm -- sh 
curl svc-multitool-nodeport
```

### LoadBalancer

- применяется в облаках, где есть балансировщик. (в microk8s можно поставить metallb)

```shell
kubectl apply -f 04-services/24_svc_lb_multitool.yaml 
kubectl get svc -n lesson3 -o wide
kubectl get ep -n lesson3
```

- создается сетевой балансировщик автоматически в клауде и можно постучаться на публичный адрес
- при этом под капотом создается сервис типа NodePort

```shell
kubectl describe svc -n lesson3 svc-multitool-lb | grep NodePort
```

## Headless и ExternalName

```shell
kubectl apply -f 04-services/25_svc_headless_external.yaml
kubectl get svc -n lesson3 -o wide
kubectl run -n lesson3 my-curl-pod --image=curlimages/curl -it --rm -- sh 
nslookup svc-external.lesson3.svc.cluster.local
nslookup svc-headless.lesson3.svc.cluster.local
```

</details>

<details>
  <summary>Lesson 4. Ingress</summary>

### Microk8s

`microk8s enable ingress` - команда включит ингресс-контроллер nginx, но не будет внешнего адреса, т.к. нет сервиса типа
LoadBalancer. Это вполне рабочая схема, но только для локального пользования

```shell
kubectl get all -n ingress
```

- создадим namespace и установим приложения

```shell
kubectl apply -f 05-ingress/apps/nginx-simple.yaml
kubectl apply -f 05-ingress/apps/multitool.yaml
```

- установим ingress

```shell
kubectl apply -f 05-ingress/01_ingress.yaml
```

- DNS имя настроено было заранее, поэтому адрес `microk8s.dens-al.ru` резолвится на одну из нод microk8s

```shell
curl http://microk8s.dens-al.ru/
curl http://microk8s.dens-al.ru/app
```

- DNS имя `example.dens-al.ru` не было заведено заранее и потому не резолвится.

```shell
curl http://example.dens-al.ru/
```

- Можно проверить командой:

```shell
 curl -H "Host: example.dens-al.ru" http://51.250.89.105
```

### Yandex Cloud K8S Cluster + Nginx Controller

- установим через Helm

```shell
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx && \
helm repo update && \
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace
```

- посмотрим, что установилось

```shell
kubectl get all -n ingress-nginx
```

- установим приложения

```shell
kubectl apply -f 05-ingress/apps/nginx-simple.yaml
kubectl apply -f 05-ingress/apps/multitool.yaml
```

- установим ingress

```shell
kubectl apply -f 05-ingress/01_ingress.yaml
```

curl -H "Host: app.dens-al.ru" http://158.160.102.82/app
</details>

<details>
  <summary>Lesson 5. Volumes</summary>

### EmptyDir

- создадим namespace, в котором будем работать

```shell
kubectl create ns lesson5
```

- установим приложения

```shell
kubectl apply -f 06-volumes/41_pod_vol.yaml
kubectl get pods -n lesson5 
sleep 5 
kubectl get pods -n lesson5 
```

- зайдем на контейнер busybox пода

```shell
kubectl exec -n lesson5 pod-emptydir -c busybox -it -- sh
```

- создадим там файл в папке

```shell
echo Hello Netology > /tmp/cache/text.txt
ls -la /tmp/cache
```

- посмотрим файл в контейнере nginx

```shell
kubectl exec -n lesson5 pod-emptydir -c nginx -it -- bash
ls -la /static
cat /static/text.txt
```

- где находятся файл text.txt?

```shell
sudo ls -la /var/lib/kubelet/pods
```

- видим, что есть много папок с именами UID. Чтобы узнать необходимую нам папку, надо узнать UID нашего пода

```shell
kubectl get pod -n lesson5 pod-emptydir -o yaml | grep uid
```

- можно записать что-то прямо из консоли ноды
- что будет если удалить под?

```shell
kubectl delete pod -n lesson5 pod-emptydir
```

### HostPath

```shell
kubectl apply -f 06-volumes/42_pod_vol.yaml
```

- зайдем на контейнер busybox пода

```shell
kubectl exec -n lesson5 pod-hostpath -c busybox -it -- sh
```

- создадим там файл в папке

```shell
echo Privet Netology > /data/privet.txt
ls -la /data
```

- что будет если удалить под?

```shell
kubectl delete pod -n lesson5 pod-emptydir
```

- если запустить под еще раз и он развернется на этой же ноде, то файлы сохранятся
- что будет, если имя volume не совпадет? 
- 
</details>