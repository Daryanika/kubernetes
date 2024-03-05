# Создание секретов
Секреты можно создавать через командные строки и манифесты

### С помощью командной строки
```
kubectl create secret generic my-user-pass --from-literal=username=<закодированный_username> --from-literal=password=<закодированный_password>
```
Закодировать можно вот таким способом
```
echo -n 'myusername' | base64
echo -n 'mypassword' | base64
```
```bXl1c2VybmFtZQ==```
```bXlwYXNzd29yZA==```

```
kubectl create secret generic my-address-param --from-literal=address=bXlhZGRyZXNz --from-literal=param=bXlwYXJhbQ==
secret/my-address-param created
```
```
echo -n 'myaddress' | base64
echo -n 'myparam' | base64
bXlhZGRyZXNz
bXlwYXJhbQ==
```

### С помощью манифеста

```
my-user-pass-secret.yaml
```

```
apiVersion: v1
kind: Secret
metadata:
  name: my-user-pass
  type: Opaque
data:
  username: <закодированный_username>
  password: <закодированный_password>
```
В данном примере
```
apiVersion: v1
kind: Secret
metadata:
  name: my-user-pass
  type: Opaque
data:
  username: bXl1c2VybmFtZQ==
  password: bXlwYXNzd29yZA==
```
Затем как обычно применяем манифест
```
kubectl apply -f my-user-pass-secret.yaml
```
Проверяем данные командой
```
kubectl get secrets
```

<img width="407" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/7b6550ef-be04-40a6-888d-58bc253e6710">

Получили секреты и с адресом, и с паролем, при этом, когда создавали манифестом, данные типа перезаписались, но так как они не обновляются, то время остается то, что раньше создано

# Использование секретов
1. Через окружение
2. Через тома

Cоздаем под, который будет использовать секреты
``` 
nano pod-using-secrets-env.yaml
 ```
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-using-secrets-env
spec:
  containers:
  - name: mycontainer
    image: nginx
    env:
      - name: USERNAME
        valueFrom:
          secretKeyRef:
            name: my-user-pass
            key: username
      - name: PASSWORD
        valueFrom:
          secretKeyRef:
            name: my-user-pass
            key: password
      - name: ADDRESS
        valueFrom:
          secretKeyRef:
            name: my-address-param
            key: address
      - name: PARAM
        valueFrom:
          secretKeyRef:
            name: my-address-param
            key: param
  restartPolicy: Never
```
```
kubectl apply -f pod-using-secrets-env.yaml
```
<img width="621" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/78a3fe88-0356-4dea-9208-4f414e63e419">

Создаем еще один под
```
nano pod-using-secrets-volume.yaml
```
```                                                      
apiVersion: v1
kind: Pod
metadata:
  name: pod-using-secrets-volume
spec:
  containers:
  - name: mycontainer
    image: nginx
    volumeMounts:                     # точка монтирования тома
    - name: user-pass-volume          # имя тома, который должен соотвествовать 
      mountPath: "/etc/user-pass"
      readOnly: true
    - name: address-param-volume
      mountPath: "/etc/address-param"
      readOnly: true
  volumes:
  - name: user-pass-volume
    secret:
      secretName: my-user-pass
  - name: address-param-volume   
    secret:
      secretName: my-address-param
  restartPolicy: Never
```

Каждый элемент **volumeMounts** указывает на том, который будет монтироваться внутри контейнера

**mountPath** путь файловой системы, куда будет монтироваться том

**readOnly** указывает, что том монтируется только для чтения

**volumes** определяет тома, которые будут подключаться

Каждый том ссылается на определенный раннее созданный секрет

**name** - имя тома в рамках одного пода

**secret** - указывает, что источником данных для тома является секрет

**secretName** - имя секрета
