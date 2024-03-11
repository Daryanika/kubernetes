# Выполнение задание 

Создание pv
```
nano postgres-pv.yaml
```
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: "/mnt/data/postgres"
```
```
kubectl apply -f postgres-pv.yaml
kubectl get pv
```
<img width="726" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/9f85396a-5e44-4aa7-8a6e-0e961b52cf7d">


Создание pvc
```
nano postgres-pv.yaml
```
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  storageClassName: manual
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
```
kubectl apply -f postgres-pvc.yaml
kubectl get pvc
```
<img width="969" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/e34abb74-da61-4679-b6e3-7b554a7b3be6">

Создаем statefulset

```
nano postgres-statefulset.yaml
```
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"   # \u0441\u0435\u0440\u0432\u0438\u0441, \u043a\u043e\u0442\u043e\u0440\u044b\u0439 \u0443\u043f\u0440\u0430\u0432\u043b\u044f\u0435\u0442 \u0434\u043e\u0441\u0442\u0443\u043f\u043e\u043c \u043a \u043f\u043e\u0434\u0443
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:latest
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          value: "exampledb"
        - name: POSTGRES_USER
          value: "exampleuser"
        - name: POSTGRES_PASSWORD
          value: "examplepassword"
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "manual"
      resources:
        requests:
          storage: 1Gi
```
```
kubectl apply -f postgres-statefulset.yaml
kubectl get statefulset
```

<img width="469" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/a11cf936-8d6f-451f-81e9-ccd4fa9dccc4">

Создание service
```
nano postgres-service.yaml
```
```
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  type: ClusterIP
  ports:
  - port: 5432
    targetPort: 5432
  selector:
    app: postgres
```
```
kubectl apply -f postgres-service.yaml
kubectl get service
```
<img width="705" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/57f77e00-031b-40a6-b2d0-a6cc27a07ca4">


