# Домашнее задание к занятию «Обновление приложений»

### Цель задания

Выбрать и настроить стратегию обновления приложения.

### Чеклист готовности к домашнему заданию

1. Кластер K8s.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Документация Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment).
2. [Статья про стратегии обновлений](https://habr.com/ru/companies/flant/articles/471620/).

-----

### Задание 1. Выбрать стратегию обновления приложения и описать ваш выбор

1. Имеется приложение, состоящее из нескольких реплик, которое требуется обновить.
2. Ресурсы, выделенные для приложения, ограничены, и нет возможности их увеличить.
3. Запас по ресурсам в менее загруженный момент времени составляет 20%.
4. Обновление мажорное, новые версии приложения не умеют работать со старыми.
5. Вам нужно объяснить свой выбор стратегии обновления приложения.

### Ответ:

1. Выбираю rolling update, т.к. обновление происходит бесшовно, и если что-то пойдет не так всегда можно откатиться к предыдущей версии.    
2. Обновление необходимо проводить в менее загруженный момент времени, и думаю что maxSurge и maxUnavailable нужно расчитывать исходя из запаса по ресурсам.    

Возможно есть вариант использовать canary, т.к. можно использовать maxSurge и maxUnavailable, чтобы избежать нехватки ресурсов, а также протестировать новую версию приложения.    

На данный момент мне проблематично наиболее точно ответить на эти вопросы, лучше всего понимание приходит на практике, когда есть возможность потрогать, поэкспериментировать и уже своими глазами увидеть результат.     

### Задание 2. Обновить приложение

1. Создать deployment приложения с контейнерами nginx и multitool. Версию nginx взять 1.19. Количество реплик — 5.
2. Обновить версию nginx в приложении до версии 1.20, сократив время обновления до минимума. Приложение должно быть доступно.
3. Попытаться обновить nginx до версии 1.28, приложение должно оставаться доступным.
4. Откатиться после неудачного обновления.

### Ответ:

1. Создем деплоймент с nginx 1.19:

```
ubuntu@master-node:~/deploy$ kubectl apply -f deploy.yml
deployment.apps/nginx unchanged
service/nginx-svc created
ubuntu@master-node:~/deploy$ kubectl get deploy -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                IMAGES                               SELECTOR
nginx   5/5     5            5           68s   nginx,network-multitool   nginx:1.19,wbitt/network-multitool   app=nginx
ubuntu@master-node:~/deploy$ kubectl get po -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE            NOMINATED NODE   READINESS GATES
nginx-66c48d986d-67dhx   2/2     Running   0          76s   10.244.1.2   worker-node-2   <none>           <none>
nginx-66c48d986d-bnltb   2/2     Running   0          76s   10.244.2.2   worker-node-1   <none>           <none>
nginx-66c48d986d-h968s   2/2     Running   0          76s   10.244.2.4   worker-node-1   <none>           <none>
nginx-66c48d986d-mbtpn   2/2     Running   0          76s   10.244.1.3   worker-node-2   <none>           <none>
nginx-66c48d986d-q25xs   2/2     Running   0          76s   10.244.2.3   worker-node-1   <none>           <none>
ubuntu@master-node:~/deploy$ kubectl describe po nginx-66c48d986d-67dhx | grep -i "Image: *nginx"
    Image:          nginx:1.19
```

2. Меняем версию на 1.20:

```
ubuntu@master-node:~/deploy$ nano deploy.yml
ubuntu@master-node:~/deploy$ kubectl apply -f deploy.yml
deployment.apps/nginx configured
service/nginx-svc unchanged
ubuntu@master-node:~/deploy$ kubectl get po -o wide
NAME                     READY   STATUS              RESTARTS   AGE     IP           NODE            NOMINATED NODE   READINESS GATES
nginx-5fddbf4656-52bgf   2/2     Running             0          17s     10.244.2.5   worker-node-1   <none>           <none>
nginx-5fddbf4656-6btpr   2/2     Running             0          5s      10.244.2.6   worker-node-1   <none>           <none>
nginx-5fddbf4656-gx2g6   2/2     Running             0          17s     10.244.1.4   worker-node-2   <none>           <none>
nginx-5fddbf4656-tbvtl   2/2     Running             0          5s      10.244.1.5   worker-node-2   <none>           <none>
nginx-5fddbf4656-tr98z   0/2     ContainerCreating   0          2s      <none>       worker-node-1   <none>           <none>
nginx-66c48d986d-67dhx   0/2     Terminating         0          7m29s   10.244.1.2   worker-node-2   <none>           <none>
ubuntu@master-node:~/deploy$ kubectl get po -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE            NOMINATED NODE   READINESS GATES
nginx-5fddbf4656-52bgf   2/2     Running   0          32s   10.244.2.5   worker-node-1   <none>           <none>
nginx-5fddbf4656-6btpr   2/2     Running   0          20s   10.244.2.6   worker-node-1   <none>           <none>
nginx-5fddbf4656-gx2g6   2/2     Running   0          32s   10.244.1.4   worker-node-2   <none>           <none>
nginx-5fddbf4656-tbvtl   2/2     Running   0          20s   10.244.1.5   worker-node-2   <none>           <none>
nginx-5fddbf4656-tr98z   2/2     Running   0          17s   10.244.2.7   worker-node-1   <none>           <none>
ubuntu@master-node:~/deploy$ kubectl describe po nginx-5fddbf4656-52bgf | grep -i "image: *nginx"
    Image:          nginx:1.20
ubuntu@master-node:~/deploy$ kubectl exec nginx-5fddbf4656-52bgf -- curl nginx-svc:9002
Defaulted container "nginx" out of: nginx, network-multitool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   139  100   139    0     0  27800      0 --:--:-- --:--:-- --:--:-- 27800
WBITT Network MultiTool (with NGINX) - nginx-5fddbf4656-gx2g6 - 10.244.1.4 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```

3. Меняем версию на 1.28, видим ошибки, но приложение доступно:

```
ubuntu@master-node:~/deploy$ kubectl apply -f deploy.yml
deployment.apps/nginx configured
service/nginx-svc unchanged
ubuntu@master-node:~/deploy$ kubectl get po -o wide
NAME                     READY   STATUS             RESTARTS   AGE   IP           NODE            NOMINATED NODE   READINESS GATES
nginx-59df66d6bd-n7dmw   1/2     ImagePullBackOff   0          8s    10.244.1.6   worker-node-2   <none>           <none>
nginx-59df66d6bd-t5k84   1/2     ImagePullBackOff   0          8s    10.244.2.8   worker-node-1   <none>           <none>
nginx-5fddbf4656-6btpr   2/2     Running            0          14m   10.244.2.6   worker-node-1   <none>           <none>
nginx-5fddbf4656-gx2g6   2/2     Running            0          14m   10.244.1.4   worker-node-2   <none>           <none>
nginx-5fddbf4656-tbvtl   2/2     Running            0          14m   10.244.1.5   worker-node-2   <none>           <none>
nginx-5fddbf4656-tr98z   2/2     Running            0          14m   10.244.2.7   worker-node-1   <none>           <none>
ubuntu@master-node:~/deploy$ kubectl get po -o wide
NAME                     READY   STATUS             RESTARTS   AGE   IP           NODE            NOMINATED NODE   READINESS GATES
nginx-59df66d6bd-n7dmw   1/2     ErrImagePull       0          30s   10.244.1.6   worker-node-2   <none>           <none>
nginx-59df66d6bd-t5k84   1/2     ImagePullBackOff   0          30s   10.244.2.8   worker-node-1   <none>           <none>
nginx-5fddbf4656-6btpr   2/2     Running            0          14m   10.244.2.6   worker-node-1   <none>           <none>
nginx-5fddbf4656-gx2g6   2/2     Running            0          14m   10.244.1.4   worker-node-2   <none>           <none>
nginx-5fddbf4656-tbvtl   2/2     Running            0          14m   10.244.1.5   worker-node-2   <none>           <none>
nginx-5fddbf4656-tr98z   2/2     Running            0          14m   10.244.2.7   worker-node-1   <none>           <none>
ubuntu@master-node:~/deploy$ kubectl exec deployment/nginx -- curl nginx-svc:9002
Defaulted container "nginx" out of: nginx, network-multitool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   139  100   139    0     0  15444      0 --:--:-- --:--:-- --:--:-- 15444
WBITT Network MultiTool (with NGINX) - nginx-5fddbf4656-tbvtl - 10.244.1.5 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)

```

4. Откатываем к предыдущей версии:

```
ubuntu@master-node:~/deploy$ kubectl rollout undo deployment nginx
deployment.apps/nginx rolled back
ubuntu@master-node:~/deploy$ kubectl get po -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE            NOMINATED NODE   READINESS GATES
nginx-5fddbf4656-6btpr   2/2     Running   0          16m   10.244.2.6   worker-node-1   <none>           <none>
nginx-5fddbf4656-bskr6   2/2     Running   0          3s    10.244.2.9   worker-node-1   <none>           <none>
nginx-5fddbf4656-gx2g6   2/2     Running   0          16m   10.244.1.4   worker-node-2   <none>           <none>
nginx-5fddbf4656-tbvtl   2/2     Running   0          16m   10.244.1.5   worker-node-2   <none>           <none>
nginx-5fddbf4656-tr98z   2/2     Running   0          16m   10.244.2.7   worker-node-1   <none>           <none>
ubuntu@master-node:~/deploy$ kubectl rollout status deployment nginx
deployment "nginx" successfully rolled out
```


## Дополнительные задания — со звёздочкой*

Задания дополнительные, необязательные к выполнению, они не повлияют на получение зачёта по домашнему заданию. **Но мы настоятельно рекомендуем вам выполнять все задания со звёздочкой.** Это поможет лучше разобраться в материале.   

### Задание 3*. Создать Canary deployment

1. Создать два deployment'а приложения nginx.
2. При помощи разных ConfigMap сделать две версии приложения — веб-страницы.
3. С помощью ingress создать канареечный деплоймент, чтобы можно было часть трафика перебросить на разные версии приложения.

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
