
Настройка внешнего подключения:
- отредактировать файл `/var/snap/microk8s/current/certs/csr.conf.template`
```shell
# [ alt_names ]
# Add
# IP.4 = 123.45.67.89
```
- обновить сертификаты `sudo microk8s refresh-certs --cert front-proxy-client.crt`.