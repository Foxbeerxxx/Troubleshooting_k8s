# Домашнее задание к занятию "`Troubleshooting`" - `Татаринцев Алексей`



### Задание 1



1. `Установка задания`
```
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml

у меня ругается, что нет namespaces web и data
```
![1](https://github.com/Foxbeerxxx/Troubleshooting_k8s/blob/main/img/img1.png)

2. `Создаю namespaces web и data`
```
kubectl create namespace web
kubectl create namespace data
и пробую снова установить
```
![2](https://github.com/Foxbeerxxx/Troubleshooting_k8s/blob/main/img/img2.png)



3. `Диагностика`

```
kubectl -n web logs deploy/web-consumer | head
kubectl -n data get svc auth-db 
```
![3](https://github.com/Foxbeerxxx/Troubleshooting_k8s/blob/main/img/img3.png)

4. `Вся беда из за того , что находятся в разных namespace`
```
Можно обращаться напрямую к  DNS-имени сервиса,для этого посмотрю его.
Сервис auth-db находится в namespace data, значит его полное имя (FQDN) в кластере:
auth-db.data.svc.cluster.local
```
![4](https://github.com/Foxbeerxxx/Troubleshooting_k8s/blob/main/img/img4.png)

5. `Чтобы заработало нужно изменить deploy`
```
kubectl -n web patch deploy web-consumer \
  --type='json' \
  -p='[{"op":"replace","path":"/spec/template/spec/containers/0/command","value":
  ["sh","-c","while true; do curl -sSf auth-db.data.svc.cluster.local; sleep 5; done"]}]'


Проверяем что podы успешно обновлены и сервис поднялся
kubectl -n web rollout status deploy/web-consumer
kubectl -n web logs deploy/web-consumer | head

```
![5](https://github.com/Foxbeerxxx/Troubleshooting_k8s/blob/main/img/img5.png)

