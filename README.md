### Домашнее задание к занятию «Хранение в K8s»
### Задание 1. Volume: обмен данными между контейнерами в поде
<details><summary>containers-data-exchange.yaml</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-exchange
  annotations:
    container1: busybox
    container2: multitool
spec:
  replicas: 1
  selector:
    matchLabels:
      app: PV
  template:
    metadata:
      labels:
        app: PV
    spec:
      containers:
      - name: busybox
        image: busybox:latest
        command: ['sh', '-c', "until false; do date >> /data/netology.txt; sleep 5; done"] 
        volumeMounts:
        - name: file
          mountPath: /data
      - name: multitool
        image: praqma/network-multitool:alpine-extra
        command: ["/bin/sh", "-c"]
        args: ["tail -f /data/netology.txt"]
        volumeMounts:
         - name: file
           mountPath: /data
      volumes:
          - name: file
            emptyDir: {}
```
</details>
описание пода с контейнерами (kubectl describe pods data-exchange)

![image](https://github.com/EremeevAN/kuber-2.1/blob/main/1.png)

вывод команды чтения файла (tail -f <имя общего файла>)

![image](https://github.com/EremeevAN/kuber-2.1/blob/main/2.png)

### Задание 2. PV, PVC

Создать Deployment приложения, состоящего из контейнеров busybox и multitool и создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.

<details><summary>pv-pvc.yaml</summary>

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local
  hostPath:
    path: /data 
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc 
spec:
  volumeName: data
  volumeMode: Filesystem
  storageClassName: "local"
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-exchange-pvc
spec:
  replicas: 1 
  selector:
    matchLabels:
      app: busybox 
  template:
    metadata:
      labels:
        app: busybox 
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ['sh', '-c', "until false; do date >> /data/netology.txt; sleep 5; done"] 
        volumeMounts:
        - name: data 
          mountPath: /data 
      - name: multitool
        image: praqma/network-multitool:alpine-extra
        command: ["/bin/sh", "-c"]
        args: ["tail -f /data/netology.txt"] 
        volumeMounts:
        - name: data 
          mountPath: /data 
      volumes:
      - name: data 
        persistentVolumeClaim:
          claimName: data-pvc 
```
</details>

Запуск

![image](https://github.com/EremeevAN/kuber-2.1/blob/main/3.png)

Продемонстрировать, что контейнер multitool может читать данные из файла в смонтированной директории, в который busybox записывает данные каждые 5 секунд.

![image](https://github.com/EremeevAN/kuber-2.1/blob/main/4.png)

Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему

Все зависит от политики persistentVolumeReclaimPolicy. Стоит значение retain, , то содержимое тома сохранится, а сам том перейдет в статус Released

![image](https://github.com/EremeevAN/kuber-2.1/blob/main/5.png)


Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV. Продемонстрировать, что произошло с файлом после удаления PV. Пояснить, почему.

![image](https://github.com/EremeevAN/kuber-2.1/blob/main/6.png)

![image](https://github.com/EremeevAN/kuber-2.1/blob/main/7.png)

По той же причине: стоит значение retain, , то содержимое тома сохранится

### Задание 3. StorageClass

Создать Deployment приложения, состоящего из контейнеров busybox и multitool и создать SC и PVC

<details><summary>sc.yaml</summary>

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local 
provisioner: microk8s.io/hostpath 
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc 
spec:
  volumeMode: Filesystem
  storageClassName: local
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-exchange-pvc
spec:
  replicas: 1 
  selector:
    matchLabels:
      app: busybox 
  template:
    metadata:
      labels:
        app: busybox 
    spec:
      containers:
      - name: busybox 
        image: busybox
        command: ['sh', '-c', "until false; do date >> /data/netology.txt; sleep 5; done"]  
        volumeMounts:
        - name: data 
          mountPath: /data 
      - name: multitool
        image: praqma/network-multitool:alpine-extra
        command: ["/bin/sh", "-c"]
        args: ["tail -f /data/netology.txt"] 
        volumeMounts:
        - name: data 
          mountPath: /data 
      volumes:
      - name: data 
        persistentVolumeClaim:
          claimName: data-pvc
```
</details>
Запуск

![image](https://github.com/EremeevAN/kuber-2.1/blob/main/8.png)

Продемонстрировать, что контейнер multitool может читать данные из файла в смонтированной директории, в который busybox записывает данные каждые 5 секунд.

![image](https://github.com/EremeevAN/kuber-2.1/blob/main/9.png)