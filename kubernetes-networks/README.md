1. Изменить readiness-пробу в манифесте deployment.yaml из прошлого Д3 на httpGet, вызывающую URL /index.html

- Внесла изменения в deployments.yaml (блок readinessProbe)
- Удалила прошлые deploy

% kubectl delete deploy --all

- Запустила заново 

% kubectl apply -f ./deployment.yaml


% kubectl get all 

NAME                                 READY   STATUS    RESTARTS   AGE
pod/hw-deployment-5647966557-66tdd   1/1     Running   0          35s
pod/hw-deployment-5647966557-czdcn   1/1     Running   0          35s
pod/hw-deployment-5647966557-wxgbl   1/1     Running   0          35s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hw-deployment   3/3     3            3           35s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/hw-deployment-5647966557   3         3         3       35s

Ready проба проходит

2. Необходимо создать манифест service.yaml, описывающий сервис типа ClusterIP, который будет направлять трафик на поды, управляемые вашим deployment

Мои поды имеют метку app=hw
Запустила deployment, проверила, что поды запустились

Применила service.yaml
% kubectl apply -f ./service.yaml   

service/hw-service created


% kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
hw-service   ClusterIP   10.104.230.186   <none>        80/TCP    6s

% kubectl get po 
NAME                             READY   STATUS    RESTARTS   AGE
hw-deployment-5647966557-66tdd   1/1     Running   0          2m9s
hw-deployment-5647966557-czdcn   1/1     Running   0          2m9s
hw-deployment-5647966557-wxgbl   1/1     Running   0          2m9s

Проверяю, что работает. Захожу в под 1:

% kubectl exec -it hw-deployment-5647966557-66tdd -- bash

root@hw-deployment-5647966557-66tdd:/# curl -s 10.104.230.186 
V1.0 - Hello world from hostname: hw-deployment-5647966557-czdcnroot@hw-deployment-5647966557-66tdd:/# 

root@hw-deployment-5647966557-66tdd:/#curl http://hw-service
V1.0 - Hello world from hostname: hw-deployment-5647966557-6g9qvroot@hw-deployment-5647966557-6g9qv:/# 

Работает.


3. Создать манифест ingress.yaml, в котором будет описан объект  типа ingress, направляющий все http запросы к хосту homework.otus на ранее созданный сервис. В результате запрос http://homework.otus/index.html должен отдавать код html страницы, находящейся в подах


% kubectl delete deploy,service --all                   

deployment.apps "hw-deployment" deleted
service "hw-service" deleted

% kubectl apply -f ./deployment.yaml                                            
deployment.apps/hw-deployment created

% kubectl apply -f ./service.yaml                                               
service/hw-service created

 % kubectl get all  
NAME                                 READY   STATUS    RESTARTS        AGE
pod/hw-deployment-5647966557-6g9qv   1/1     Running   1 (5m12s ago)   47h
pod/hw-deployment-5647966557-77k4p   1/1     Running   1 (5m12s ago)   47h
pod/hw-deployment-5647966557-tlspd   1/1     Running   1 (5m12s ago)   47h

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/hw-service   ClusterIP   10.111.29.153   <none>        80/TCP    47h

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hw-deployment   3/3     3            3           2d

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/hw-deployment-5647966557   3         3         3       2d

% kubectl apply -f ./ingress.yaml 

ingress.networking.k8s.io/hw-ingress created


% kubectl get ingress
NAME         CLASS   HOSTS           ADDRESS        PORTS   AGE
hw-ingress   nginx   homework.otus   192.168.49.2   80      3m29s

% sudo nano /etc/hosts
##
127.0.0.1       localhost
192.168.49.2 homework.otus


(base) dinakotlyar@192 kubernetes-networks % kubectl exec -it hw-deployment-5647966557-6g9qv -- bash
Defaulted container "web-server" out of: web-server, init-container (init)
root@hw-deployment-5647966557-6g9qv:/# curl homework.otus
V1.0 - Hello world from hostname: hw-deployment-5647966557-6g9qvroot@hw-deployment-5647966557-6g9qv:/# 

Вызов из пода по хосту работает. 
