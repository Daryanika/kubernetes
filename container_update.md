# Задание на семинар 
Создать под с двумя контейнерами внутри (любыми - связность и корректность работы в данном случае не важна). Произвести процедуру обновления из 3 версий:
— Обновить только первый контейнер
— Обновить только второй контейнер
— Обновить оба контейнера.

# Выполнение 
Чтобы работать с подами, необходимо создать его через конфигурационный файл
```
nano mypod.yaml
```
Внутри которого указываются данные о поде
```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx-container
    image: nginx:latest           
    ports:
    - containerPort: 80
  - name: redis-container
    image: redis:latest
    ports:
    - containerPort:6700
```
После создания файл применяется
```
kubectl apply -f mypod.yaml
```
Проверка
```
kubectl get pods
```
<img width="420" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/50666ec1-08e3-45a8-9ce7-75e312e2215e">

Командой 
```
kubectl describe pod my-pod
```
можно увидеть более детальную информацию
<img width="1050" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/d25157eb-1e34-4bc6-8de4-241623980083">


### Обновление 1
Просто исправляем конфигурационный файл и делаем так, как нам нужно. В данном случае исправим версию контейнера nginx
```
nano mypod.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx-container
    image: nginx:1.19.0  # обновленная версия         
    ports:
    - containerPort: 80
  - name: redis-container
    image: redis:latest
    ports:
    - containerPort:6700
```
Применяем файл и проверяем, что версия обновилась
```
kubectl apply -f mypod.yaml
```
Проверка
```
kubectl get pods
```
<img width="428" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/80cb1749-8080-43e4-ba0b-1ea32a8fc5d5">
Командой 
```
kubectl describe pod my-pod
```
можно увидеть более детальную информацию
<img width="1080" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/1b01b1ce-ae94-45cb-bafb-c6a250298c78">
Можно заметить, что версия nginx поменялась, значит обновление прошло успешно


ДОПОЛНЮ ПОЗЖЕ
