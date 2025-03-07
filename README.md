# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к приложению, установленному в предыдущем ДЗ и состоящему из двух контейнеров, по разным портам в разные контейнеры как внутри кластера, так и снаружи.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.
2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.
3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.




1) Создадим deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: dep-multitool
  name: dep-multitool
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: multitool
  template:
    metadata:
      labels:
        app: multitool
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool:latest
        ports:
        - containerPort: 8080
        env:
        - name: HTTP_PORT
          value: "8080"
```


2) Создадим Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080

```
apiVersion: v1
kind: Service
metadata:
  name: svc-multitool
spec:
  selector:
    app: multitool
  ports:
    - name: for-nginx
      port: 9001
      targetPort: 80
    - name: for-multitool
      port: 9002
      targetPort: 8080
```


3) Создадим отдельный Pod с приложением multitool и убедимся с помощью curl, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: pod-multitool
  name: pod-multitool
  namespace: default
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool:latest
    ports:
      - containerPort: 8090
        name: multitool-port
```


![alt text](https://github.com/MaratKN/kuber-homeworks-04/blob/main/1.png)

```
root@DebianNew:~/.kube# kubectl exec pod-multitool -- curl 10.1.83.195:80
root@DebianNew:~/.kube# kubectl exec pod-multitool -- curl 10.1.83.196:80
root@DebianNew:~/.kube# kubectl exec pod-multitool -- curl 10.1.83.197:80
```
![alt text](https://github.com/MaratKN/kuber-homeworks-04/blob/main/2.png)
![alt text](https://github.com/MaratKN/kuber-homeworks-04/blob/main/3.png)


4) Проверим доступ с помощью curl по доменному имени сервиса.
```
root@DebianNew:~/.kube# kubectl exec pod-multitool -- curl svc-multitool.default.svc.cluster.local:9001
```

![alt text](https://github.com/MaratKN/kuber-homeworks-04/blob/main/4.png)

------

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.
2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.


1) Создадим сервис service-dep-nodeport.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: svc-multitool-nodeport
spec:
  selector:
    app: multitool
  ports:
    - name: for-nginx
      nodePort: 30500
      targetPort: 80
      port: 80
    - name: for-multitool
      nodePort: 30501
      port: 8080
      targetPort: 8080
  type: NodePort
```

2) Проверим доступ с помощью браузера или curl с локального компьютера.

![alt text](https://github.com/MaratKN/kuber-homeworks-04/blob/main/5.png)
![alt text](https://github.com/MaratKN/kuber-homeworks-04/blob/main/6.png)


------

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
