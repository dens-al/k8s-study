# RBAC
Role-based access control (RBAC) - управление доступом основанное на ролях.

Для настройки таких доступов в Kubernetes необходимо создать следующие артефакты:
- Пользователь user, или service account (ServiceAccount)
- Role, ClusterRole
- RoleBinding, ClusterRoleBinding

- [Пользователи и авторизация RBAC в Kubernetes](https://habr.com/ru/company/flant/blog/470503/)
- [RBAC with Kubernetes in Minikube](https://medium.com/@HoussemDellai/rbac-with-kubernetes-in-minikube-4deed658ea7b)
## Создание пользователя

```shell
openssl genrsa -out jean.key 2048
```
Запрос на подпись сертификата (certificate signing request, CSR). CN — имя пользователя, O — группа. Можно устанавливать разрешения по группам. Это упростит работу, если, например, у вас много пользователей с одинаковыми полномочиями:
### Без группы
```shell
openssl req -new -key vasya.key \
-out vasya.csr \
-subj "/CN=vasya"
```
### С группой под названием *dev*
```shell
openssl req -new -key vasya.key \
-out vasya.csr \
-subj "/CN=vasya/O=dev"
```
### Если пользователь входит в несколько групп
```shell
openssl req -new -key vasya.key \
-out vasya.csr \
-subj "/CN=vasya/O=dev/O=devops"
```

Подпишите CSR в Kubernetes CA. Мы должны использовать сертификат CA и ключ, которые обычно находятся в /etc/kubernetes/pki. Сертификат будет действителен в течение 500 дней:
```shell
openssl x509 -req -in vasya.csr \
-CA /etc/kubernetes/pki/ca.crt \
-CAkey /etc/kubernetes/pki/ca.key \
-CAcreateserial \
-out vasya.crt -days 500
```

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: petya-csr
spec:
  groups:
  - system:authenticated
  request: {BASE64_csr}
  expirationSeconds: 86400  # one day
  usages:
  - client auth
  signerName: kubernetes.io/kube-apiserver-client
```

------
```shell
kubectl create sa netology
kubectl create token netology
kubectl apply -f secret.yaml
kubectl describe secret token
export TOKEN=""
curl --cacert 08-rbac/ca.crt --header "Authorization: Bearer ${TOKEN}" -X GET https://158.160.96.174:16443/api 
```

## Create user
openssl genrsa -out test.key 2048
# create CSR
openssl req -new -key test.key -out test.csr -subj "/CN=test/O=ops"
# sign CSR
openssl x509 -req -in test.csr -CA 08-rbac/ca.crt -CAkey 08-rbac/ca.key -CAcreateserial -out test.crt -days 10
# create k8s config
kubectl config set-credentials test --client-certificate test.crt --client-key test.key --embed-certs=kubecttrue
kubectl config set-context test --cluster=microk8s-cluster --user=test


###
kubectl get csr
kubectl certificate approve ssl-csr
kubectl get csr ssl-csr -o jsonpath={.status.certificate} | base64 --decode > cert.crt



