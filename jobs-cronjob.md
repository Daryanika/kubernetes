# Задание 
Создать простой CronJob, с расписанием на выполнение раз в 5 минут. Поставить время выполнения (busybox с sleep 600) 10 минут, чтобы выполнение задачи длилось достаточное количество времени.

Что нужно сделать:
— Найти способ досрочно запустить запланированную задачу средствами Кубера.
— Удалить любой произвольный запуск CronJob
— Использовать все изученные политики перезапуска в CronJob:
Allow
Forbid
Replace.

# Выполнение 
Создаем конфиг файл для джоба
```
nano my-cronjob.yaml
```
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: my-busybox
            image: busybox
            args:
            - /bin/sh
            - -c
            - sleep 600
          restartPolicy: OnFailure
```
<img width="634" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/47a9a627-a027-4ac7-8733-a63db44243c5">

Можно проверить, что кронджоб создал свои джоб для выполнения задачи 
<img width="488" alt="image" src="https://github.com/Daryanika/kubernetes/assets/147329314/201910ae-ecb5-4a9b-8736-367300570ec0">

Создаем второй кронджоб 
```
nano cronjob.yaml
```
```
 
```


