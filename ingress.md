Создать кластер БД из трех сущностей (как это делалось уроками ранее). Создать сервис, через который далее будет происходить работа с БД и опубликовать его с помощью ingress. Далее подключиться к БД через внешний адрес и убедиться, что все работает.

# Выполнение
создание секретов для базы данных 
```
nano mysql-secret.yaml
```
```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  mysql-root-password: cm9vdF9wYXNzd29yZA== # base64 для "root_password"
  mysql-user: bXlzcWx1c2Vy # base64 для "mysqluser"
  mysql-password: bXlzcWxwYXNz # base64 для "mysqlpass"
```
```
kubectl apply -f mysql-secret.yaml
kubectl get secret
```
<img width="551" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/7064ae9f-f0b1-43d3-a7cf-c744e1426d83">

Cоздание стэйтфулсета
```
nano mysql-statefulset.yaml
```
```
 apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  serviceName: "mysql"
  replicas: 1
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:latest
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: mysql-root-password
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: mysql-user
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: mysql-password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-persistent-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
В этом же манифесте создается pvc
```
kubectl apply -f mysql-statefulset.yaml
kubectl get pods -l app=mysql
```
<img width="558" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/06eee54d-f666-44f1-b3a3-c75371544242">

Создадим pv
```
nano mysql-pv.yaml
```
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: "/mnt/data/mysql"
```
Необходимо сделать директорию для точки монтирования
```
sudo mkdir -p /mnt/data/mysql
```
После создания директории для монтирования можно уже применить pv
```
kubectl apply -f mysql-pv.yaml
```
Создаем сервис для базы данных 
```
nano mysql-service.yaml
```
```
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  type: ClusterIP
  ports:
  - port: 3306
    targetPort: 3306   # порт пода
  selector:
    app: mysql
```
```
kubectl apply -f mysql-service.yaml
```

Делаем сам ингресс
```
minikube addons enable ingress
```
Проверить ингресс, которые есть в наличии 
```
kubectl get ingress
```
Создаем манифест
```
nano mysql-ingress.yaml
```
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mysql-ingress
spec:
  rules:
  - http:
      paths:
      - path: /mysql
        pathType: Prefix
        backend:
          service:
            name: mysql-service
            port:
              number: 3306
```
```
kubectl apply -f mysql-ingress.yaml
kubectl describe ingress mysql-ingress
```
```
kubectl port-forward svc/mysql-service 3306:3306
```




