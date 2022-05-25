# Домашнее задание к занятию "13.3 работа с kubectl"
## Задание 1: проверить работоспособность каждого компонента

<details>

  <summary>Описание задачи</summary>  
  
Для проверки работы можно использовать 2 способа: port-forward и exec. Используя оба способа, проверьте каждый компонент:
* сделайте запросы к бекенду;
* сделайте запросы к фронту;
* подключитесь к базе данных.

</details>

### Решение

##### Подготовка
Создадим неймспейс:
>kubectl create namespace 

Задеплоим приложение:
>kubectl apply -f ./app/

##### port-forward

* сделайте запросы к бекенду  

Делаем форвард:
```bash
kubectl port-forward -n devkub-13-03 service/prod-back-srv 9001:9000
Forwarding from 127.0.0.1:9001 -> 9000
Forwarding from [::1]:9001 -> 9000
```

Проверяем:
```bash
curl http://localhost:9001/api/news/
[{"id":1,"title":"title 0","short_description":"small text 0small text 0small text 0small text 0small 
```


* сделайте запросы к фронту  

Делаем форвард:
```
kubectl port-forward -n devkub-13-03 service/prod-front-srv 8081:8080
Forwarding from 127.0.0.1:8081 -> 80
Forwarding from [::1]:8081 -> 80
Handling connection for 8081
```  

Проверяем:
```bash
curl http://localhost:8081
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" href="/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite App</title>
    <script type="module" crossorigin src="/assets/index.7c2bfd2c.js"></script>
    <link rel="stylesheet" href="/assets/index.038ca730.css">
  </head>
  <body>
    <div id="app"></div>
    
  </body>
</html>
```
* подключитесь к базе данных  

Делаем форвард:
```
kubectl port-forward -n devkub-13-03 service/prod-db-srv 5432:543232
Forwarding from 127.0.0.1:5432 -> 5432
Forwarding from [::1]:5432 -> 5432
```

Проверяем:
```
pg_isready -d news -h localhost -p 5432 -U postgres  
localhost:5432 - accepting connections
```

##### exec  

Получаем имена подов:
```
kubectl get pods -n devkub-13-03
NAME                     READY   STATUS    RESTARTS   AGE
back-59c69d7bcd-qjgzk    1/1     Running   0          51m
db-0                     1/1     Running   0          51m
front-76965c974d-7mzr2   1/1     Running   0          51m
```

* сделайте запросы к бекенду;
```bash
kubectl exec -it -n devkub-13-03 back-59c69d7bcd-qjgzk -- curl http://localhost:9000/api/news/
[{"id":1,"title":"title 0","short_description":"small text 0small text 0small text 0small text 0small text 0small text 0small text 0small text 0small text 0small text 0","preview":"/static/image.png"},{"id":2,"title":"title 1","short_description":"small text 1small text 1small text 1small text 1small text 1small text 1small text 1small text 1small text 1small text 1","preview":"/static/image.png"},{"id":3,"t
```

* сделайте запросы к фронту;
```bash
kubectl exec -it -n devkub-13-03 front-76965c974d-7mzr2 -- curl http://localhost:80st:80
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" href="/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite App</title>
    <script type="module" crossorigin src="/assets/index.7c2bfd2c.js"></script>
    <link rel="stylesheet" href="/assets/index.038ca730.css">
  </head>
  <body>
    <div id="app"></div>
    
  </body>
</html>

```

* подключитесь к базе данных.
```bash
kubectl exec -it -n devkub-13-03 db-0 -- pg_isready -d news -h localhost -p 5432 -U postgres 
localhost:5432 - accepting connections
```


---

## Задание 2: ручное масштабирование

<details>

  <summary>Описание задачи</summary>  
  
При работе с приложением иногда может потребоваться вручную добавить пару копий. Используя команду kubectl scale, попробуйте увеличить количество бекенда и фронта до 3. Проверьте, на каких нодах оказались копии после каждого действия (kubectl describe, kubectl get pods -o wide). После уменьшите количество копий до 1.

</details>

### Решение

Проверяем текущее состояние:
```bash
kubectl get pods -o wide -n devkub-13-03
NAME                     READY   STATUS    RESTARTS   AGE   IP              NODE    NOMINATED NODE   READINESS GATES
back-59c69d7bcd-qjgzk    1/1     Running   0          60m   10.233.90.203   node1   <none>           <none>
db-0                     1/1     Running   0          60m   10.233.94.167   node0   <none>           <none>
front-76965c974d-7mzr2   1/1     Running   0          60m   10.233.90.204   node1   <none>           <none>
```

Увеличиваем количество копий фронта и бекенда:
```
kubectl scale --replicas=3 deploy/back -n devkub-13-03
deployment.apps/back scaled

kubectl scale --replicas=3 deploy/front -n devkub-13-03
deployment.apps/front scaled
```

Проверяем результат, смотрим на каких нодах запустились копии:
```
kubectl get pods -o wide -n devkub-13-03
NAME                     READY   STATUS    RESTARTS   AGE   IP              NODE    NOMINATED NODE   READINESS GATES
back-59c69d7bcd-72bkh    1/1     Running   0          31s   10.233.94.168   node0   <none>           <none>
back-59c69d7bcd-kqvcg    1/1     Running   0          31s   10.233.90.205   node1   <none>           <none>
back-59c69d7bcd-qjgzk    1/1     Running   0          64m   10.233.90.203   node1   <none>           <none>
db-0                     1/1     Running   0          64m   10.233.94.167   node0   <none>           <none>
front-76965c974d-7mzr2   1/1     Running   0          64m   10.233.90.204   node1   <none>           <none>
front-76965c974d-p2l8h   1/1     Running   0          13s   10.233.90.206   node1   <none>           <none>
front-76965c974d-q779s   1/1     Running   0          13s   10.233.94.169   node0   <none>           <none>
```

---
