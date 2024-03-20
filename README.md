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

</details>

<details>
  <summary>Lesson 6. PV, PVC, SC</summary>

### PersistentVolume

- создадим namespace, в котором будем работать

```shell
kubectl create ns lesson6
```

```shell
kubectl apply -f 06-volumes/50_pod_manual.yaml
```

- включим hostpath-storage на microk8s `microk8s enable hostpath-storage`. Этот SC будет по умолчанию

```shell
kubectl get sc
```

- проверим на примере, не указывая StorageClass

```shell
kubectl apply -f 06-volumes/51_pod-pvc-local.yaml
```

- включим nfs на microk8s `microk8s enable nfs`. Кроме того, надо установить пакет nfs-common на всех
  нодах `sudo apt update && sudo apt install -y nfs-common`

```shell
kubectl get sc
```

- создадим pod с PVC из nfs

```shell
kubectl apply -f 06-volumes/52_pod-pvc-nfs.yaml
```

- включим ceph используя microceph
  по [инструкции](https://www.virtualizationhowto.com/2023/08/kubernetes-persistent-volume-setup-with-microk8s-rook-and-ceph/)

```shell
kubectl get sc
```

- создадим pod с PVC из Ceph

```shell
kubectl apply -f 06-volumes/53_pod-pvc-ceph.yaml
```

</details>

<details>
  <summary>Lesson 7. ConfigMap, Secret</summary>

### ConfigMap Secret

- кодирование и докодирование в base64

```shell
echo netology | base64
echo bmV0b2xvZ3kK | base64 -d
```

- создадим очередной namespace

```shell
kubectl create ns lesson7
```

- заведем configMap и secret

```shell
kubectl apply -f 07-configs_secrets/60_configmap_secret.yaml
```

- применим конфигурацию с configMap и secret в виде volume

```shell
kubectl apply -f 07-configs_secrets/61_multitool_vol.yaml
```

- получим новые объекты: configMap и secret

```shell
kubectl get configmaps -n lesson7
kubectl describe configmaps -n lesson7 my-configmap
kubectl get secrets -n lesson7
kubectl describe secrets -n lesson7 my-secret
kubectl get secrets -n lesson7 my-secret -o yaml
```

- появились файлы с именами ключей

```shell
kubectl exec -n lesson7 multitool-vol -it -- tree /etc/config/config
kubectl exec -n lesson7 multitool-vol -it -- tree /etc/config/secret
```

- применим конфигурацию с теми же configMap в виде env

```shell
kubectl apply -f 07-configs_secrets/62_multitool_env.yaml
```

- получены переменные среды

```shell
kubectl exec -n lesson7 multitool-env -it -- env
```

- применим конфигурацию с теми же configMap в виде env_from

```shell
kubectl apply -f 07-configs_secrets/63_multitool_env_all.yaml
```

- видим все значения переменных окружения

```shell
kubectl exec -n lesson7 multitool-env-all -it -- env
```

- поменяем в значения в configMap и применяем и посмотрим, что произойдет

### Nginx config and htpasswd

- файл конфигурации в виде configMap и пароль от htpasswd в виде secret

```shell
kubectl apply -f 07-configs_secrets/64_nginx.yaml
```

- можно создавать секреты в консоли

```shell
kubectl create secret
```

```shell
kubectl create configmap -h
```

</details>

<details>
  <summary>Lesson 8. RBAC</summary>

- создадим очередной namespace

```shell
kubectl create ns lesson8
```

### Service Account

- при создании namespace создается SA по умолчанию, при этом к подам k8s по умолчанию будет подключать данный SA

```shell
kubectl get sa -n lesson8
```

- создадим собственный SA, убедимся, что у него ничего нет и создадим и токен

```shell
kubectl create sa netology-sa -n lesson8
kubectl describe sa -n lesson8 netology-sa
```

- сгенерируем временный токен для нашего SA

```shell
kubectl create token netology-sa -n lesson8
```

- этот токен можно использовать для подключения к кластеру. Также нам потребуется ca.crt, который можно скачать с
  кластера (в microk8s сертификаты лежат в пути `/var/snap/microk8s/current/certs`)

```shell
export TOKEN=<token>
curl --cacert 08-rbac/ca.crt --header "Authorization: Bearer ${TOKEN}" -X GET https://<IP_address>:16443/api 
```

- то, что данный токен временный можно убедиться расшифровав его на сайте [jwt.io](https://jwt.io/). Видим, что
  expiration time через 1 час
- для того, чтобы иметь постоянный токен, необходимо сделать объект secret

```shell
kubectl apply -f 08-rbac/80_secret.yaml
kubectl describe sa netology-sa -n lesson8
```

- теперь можно создать под с использованием этого SA и обратиться к API кластера из пода

```shell
kubectl apply -f 08-rbac/81_pod_multitool_sa.yaml
kubectl exec -n lesson8 pod-multitool-default -- ls -la /var/run/secrets/kubernetes.io/serviceaccount
```

- теперь можно использовать этот токен для запроса к API.

```shell
kubectl exec -n lesson8 pod-multitool-default -it -- bash
curl --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
    --header "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
    -X GET https://<IP_address>:16443/api
```

------

### User Account

- создадим сертификат вручную с именем `test` и группой `ops` и подпишем CA

```shell
openssl genrsa -out test.key 2048
openssl req -new -key test.key -out test.csr -subj "/CN=test/O=ops"
openssl x509 -req -in test.csr -CA 08-rbac/ca.crt -CAkey 08-rbac/ca.key -CAcreateserial -out test.crt -days 10
cat test.crt
```

- получим сертификат, который можно использовать для подключения к кластеру

```shell
kubectl config set-credentials test_user --client-certificate test.crt --client-key test.key --embed-certs=true
kubectl config view
```

- создадим новый контекст к текущему кластеру с новыми кредами

```shell
kubectl config set-context test_connection --cluster=microk8s-cluster --user=test_user
kubectl config use-context test_connection
kubectl get nodes
```

- то же самое можно делать с помощью API k8s и объекта типа `CertificateSigningRequest`
- для этого надо сгенерировать CSR и отправить в кластер запрос

```shell
openssl genrsa -out test2.key 2048
openssl req -new -key test2.key -out test2.csr -subj "/CN=test2/O=ops"
cat test2.csr | base64
```

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: super-csr
spec:
  groups:
    - system:authenticated
  request: { BASE64_csr }
  expirationSeconds: 86400  # one day
  usages:
    - client auth
  signerName: kubernetes.io/kube-apiserver-client
```

```shell
kubectl apply -f 08-rbac/82_csr.yaml
kubectl get csr
```

- аппрувнем запрос

```shell
kubectl certificate approve super-csr
kubectl get csr
```

- заберем сертификат

```shell
kubectl get csr super-csr -o jsonpath={.status.certificate} | base64 --decode
```

- теперь можно создать конфигурацию для подключения к кластеру с полученным сертификатом

```shell
kubectl get csr super-csr -o jsonpath={.status.certificate} | base64 --decode
```

### Role, RoleBinding

- дадим пользователю права на просмотр подов и деплойментов

```shell
kubectl apply -f 08-rbac/83_role_rolebind.yaml
```

- список api ресурсов

 ```shell
kubectl api-resources
```

- дадим группе права описанные ранее

```shell
kubectl apply -f 08-rbac/84_rolebind.yaml
```

- переключимся на контекст пользователя и убедимся, что права есть

</details>


<details>
  <summary>Lesson 9. Helm</summary>

### Helm as Packet Manager

- список всех подключенных репозиториев

```shell
helm repo list
```

- имеется поиск приложений по всем репозиториям

```shell
helm search repo nginx
```

- установим nginx из репозитория bitnami

```shell
helm install web bitnami/nginx
```

- список установленных приложений по всем namespace

```shell
helm list -A
```

- эмуляция установки

```shell
helm install web bitnami/nginx --dry-run
```

```shell
helm status web
```

```shell
helm upgrade --install web bitnami/nginx
```

```shell
helm upgrade --install web bitnami/nginx --namespace web
```

```shell
helm upgrade --install web bitnami/nginx --namespace web --create-namespace
```

```shell
helm pull bitnami/nginx
```

</details>

<details>
  <summary>Lesson 12. Network in k8s</summary>

### Таблица маршрутизации

- таблица на нодах

```shell script
route -n
ip -a
ip -r
```

- таблица на flannel

```shell script
brctl show
```

- установим какой-нибудь деплоймент на большое кол-во реплик

```shell
kaf 02-deployments/32_dpl_svc_multitool.yaml
```

-
- посмотрим маршруты и сетевые интерфейсы на нодах

```shell
route -n
ip -a
ip -r
brctl show
kubectl get po -o wide
```

- kube-proxy создает правила iptables

```shell script
iptables -t nat -vnL
# Удаление всех правил iptables. Не выполнять на production кластерах
iptables -F
# Через некоторое время правила будут созданы заново
iptables -t nat -vnL
```

- примерное количество подов запущенных на ноде:

```shell script
ip a | grep veth | wc -l
```

- можно сверить

```shell script
kubectl get po -A -o wide | grep node1 | wc -l
```

### Network policies

- создадим под

```shell
kubectl run nginx --image=nginx --labels="app=web"
kubectl expose pod/nginx --port=80
sleep 20
kubectl get pods --show-labels
```

```shell
kubectl run my-curl-pod --image=curlimages/curl -it --rm -- sh 
```

- установим политику по умолчанию запретить все

```shell
kaf 11-networking/network-policy.yaml
```

- проверим что доступа нет

```shell
kubectl run my-curl-pod --image=curlimages/curl -it --rm -- sh
```

- проверим, что доступ появился

```shell
kubectl run my-curl-pod --image=curlimages/curl --labels="app=curl" -it --rm -- sh
```

</details>
