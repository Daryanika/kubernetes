# Создание нового пространства имен

Для начала работы необходимо создать новое пространство имен с помощью конфигурационного файла

```
vi my-namespace.yaml
```
При заполнении файла необходимо указать версию кубернетиса, сущность, которая будет создана по этому файлу, а также дополнительные данные
```
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace 
```
После создание и сохранения файла необходимо его применить для создания нового пространства имен

```
kubectl apply -f my-namespace.yaml
```

Для проверки, что пространство имен действительно сформировалось, можно запустить команду 
```
kubectl get namespaces
```


# Взаимодейтсвие подов и реплик с одинаковыми метками 

В случае, когда создается под, а затем такие же реплики, то они сливаются вместе, составляя в сумме количество, указанное в конфигурационном файле реплик

Для создания подов и реплик можно использовать конфигурационный файл.
Создание пода:
```
vi my-pod.yaml
```
Внутри указываем сущность под и остальные данные (например, пространство имен, которое будет использоваться)
```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: my-namespace
  labels:
    app: my-new-app     # уникальный тег для идентификации пода
spec:
  containers:
  - name: my-container
    image: nginx           
    ports:
    - containerPort: 80
```
Для создания и применения используется так же:
```
kubectl apply -f my-pod.yaml
```
Для просмотра созданных подов подойдет функция 
```
kubectl get pods
```
При желании посмотреть только поды, которые относятся к определенному пространству имен, нужно добавить флаг
```
kubectl get pods -n my-namespace
```

Создание и применеие реплик происходит практически таким же образом, поэтому приведен только код
```
vi my-replica.yaml
```
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replica
  namespace: new-namespace
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: my-new-app
  template:                      
    metadata:
      labels:
        app: my-new-app
    spec:
      containers:
      - name: my-container
        image: nginx
        ports:
        - containerPort: 80
```
```
kubectl apply -f my-replica.yaml
```

Чтобы проверить, что поды и реплики дейтствительно объединились, достаточно посмотреть поды
```
kubectl get replicaset -n my-namespace
```
```
kubectl get pods -n my-namespace
```

# Примеры выполнения 
<img width="552" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/15d01666-dbb4-4c64-a473-ce199bf336aa">


<img width="538" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/2ba2219f-7a34-43a5-ac39-5fa175e24087">
