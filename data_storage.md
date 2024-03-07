Проверка кластера
```
kubectl cluster-info
```

Создание SC 
```
nano custom-storage-class.yaml
```
Внутри как обычно
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: custom-storage-class
provisioner: k8s.io/minikube-hostpath    #указывает какой драйвер хранилища используется
reclaimPolicy: Delete                     # что происходит с PV, когда будет удаляться PVC
volumeBindingMode: Immediate             # когда и как PV привязывается к PVC
```

Применение как всегда 
```
kubectl apply -f custom-storage-class.yaml
```
И просмотр 
```
kubectl get sc
```
<img width="1046" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/b989e81c-f4a2-4994-b869-864b8ff9a813">

более подробно можно посмотреть командой
```
kubectl describe sc custom-storage-class
```

Создаем PV
```
nano db-pvc.yaml
```
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: custom-storage-class # определяем, что будет использоваться для создания
```
```
nano write-pvc.yaml
```
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: write-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: custom-storage-class
```
Применение файлов
```
kubectl apply -f db-pvc.yaml
kubectl apply -f write-pvc.yaml
```
Просмотр созданных PVC
```
kubectl get pvc
```
Проверка PV
```
kubectl get pv
```
При создании PVC кубер автоматически провиженит PV с использованием указанного класса
<img width="1353" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/6a8cc3c0-8ada-4f21-bc27-9ad5048e70b8">

# использование вместе с БД
```
nano postgres-pod.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: postgres-pod
spec:
  containers:
  - name: postgres-container
    image: postgres:latest
    env:
    - name: POSTGRES_DB
      value: "exampledb"
    - name: POSTGRES_USER
      value: "exampleuser"
    - name: POSTGRES_PASSWORD
      value: "examplepassword"
    volumeMounts:    # точка монтирования для контейнера
    - mountPath: "/var/lib/postgresql/data"
      name: postgres-storage
  volumes:    # определяется том, который будет связан с PVC 
  - name: postgres-storage 
    persistentVolumeClaim:
      claimName: db-pvc  # то, с чем нужно связать
```
```
kubectl apply -f postgres-pod.yaml
```
```
kubectl get pods
```
<img width="467" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/046d6bb9-a063-4f6e-a651-70b00c02f33b">


# Заполнение БД
Развертываем таблицу из контейнера и инициализировать ее данными через PVC
Создание таблицы
```
kubectl exec postgres-pod --stdin --tty -- psql -U exampleuser -d exampledb -c "CREATE TABLE example_table (id SERIAL PRIMARY KEY, data VARCHAR(100));"
```
Вставка данных 
```
kubectl exec postgres-pod --stdin --tty -- psql -U exampleuser -d exampledb -c "INSERT INTO example_table (data) VALUES ('Hello, Kubernetes!');"
```
Проверка данных из таблицы
```
kubectl exec postgres-pod --stdin --tty -- psql -U exampleuser -d exampledb -c "SELECT * FROM example_table;"
```
<img width="1234" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/3fdf2652-1587-40ae-9adc-a7305e6bc6a1">

# Выполнение ДЗ
```
nano write-to-file-pod.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: write-to-file-pod
spec:
  containers:
  - name: busybox-container
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do date >> /data/output.txt; sleep 5; done"]
    volumeMounts:
    - name: data-storage
      mountPath: "/data"   # точка монтирования, куда подключается том с данными
  volumes:
  - name: data-storage
    persistentVolumeClaim:
      claimName: write-pvc
```
```
kubectl apply -f write-to-file-pod.yaml
```
<img width="500" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/2aaeda6b-05f3-479b-ae09-6d10959f5b9d">
Просматриваем файл из пода

```
kubectl exec write-to-file-pod -- cat /data/output.txt
```

<img width="709" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/db647096-fbe4-45de-839c-99168666f08d">

Для смены PV необходимо выполнить следующие шаги:

1. отсоединение/удаление пода от PVC, при этом PV останется
```
kubectl delete pod write-to-file-pod
```
2. Создание нового пода

```
nano read-from-file-pod.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: read-from-file-pod
spec:
  containers:
  - name: busybox-container
    image: busybox
    command: ["/bin/sh", "-c", "sleep 10000"]
    volumeMounts:
    - name: data-storage
      mountPath: "/data"
  volumes:
  - name: data-storage
    persistentVolumeClaim:
      claimName: write-pvc
```
```
kubectl apply -f read-from-file-pod.yaml
```
```
```
Проверяем данные, которые были записаны в предыдущем поде, сохранились

```
kubectl exec read-from-file-pod -- cat /data/output.txt
```
<img width="757" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/5a60f79f-18ed-4daa-9eef-69ab6d08da5e">


# Повторное подключение к базе данных 
Удаление предыдущей БД

```
kubectl delete pod postgres-pod
```
Создаем новый под

```
nano postgres-pod-reconnect.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: postgres-pod-reconnect
spec:
  containers:
  - name: postgres-container
    image: postgres:latest
    env:
    - name: POSTGRES_DB
      value: "exampledb"
    - name: POSTGRES_USER
      value: "exampleuser"
    - name: POSTGRES_PASSWORD
      value: "examplepassword"
    volumeMounts:
    - mountPath: "/var/lib/postgresql/data"
      name: postgres-storage
  volumes:
  - name: postgres-storage
    persistentVolumeClaim:
      claimName: db-pvc
```
```
kubectl apply -f postgres-pod-reconnect.yaml
```
<img width="569" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/bf58b387-969f-4a31-8fbb-ac1b5b082286">
Проверяем, что данные так же доступны

```
kubectl exec postgres-pod-reconnect --stdin --tty -- psql -U exampleuser -d exampledb -c "SELECT * FROM example_table;"
```

<img width="1298" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/d09ba806-ac03-4a02-beca-452ddbd92962">



