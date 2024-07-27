
1. Создать манифест pvc.yaml, описывающий PersistentVolumeClaim, запрашивающий хранилище с storageClass по умолчанию

% kubectl apply -f ./pvc.yaml
persistentvolumeclaim/hw-pvc created

% k9s
:pvc

NAME          STATUS           VOLUME                                           CAPACITY            ACCESS MODES     STORAGECLASS                           
hw-pvc        Bound            pvc-fc4e1925-820d-4eb5-9d9d-d88d3a8921b6         1Gi                 RWO              standard                       


2. Создать манифест ст.yaml для объекта типа configMap, описывающий произвольный набор пар ключ-значение


% kubectl apply -f ./cm.yaml 
configmap/hw-configmap created

% k9s

:cm

NAME                                 DATA                AGE 
hw-configmap                         3                   61s      

Describe hw-configmap

Name:         hw-configmap                                                                                                                                                   │
Namespace:    homework                                                                                                                                                       │
Labels:       <none>                                                                                                                                                         │
Annotations:  <none>                                                                                                                                                         │

Data                                                                                                                                                                ====                                                                                                                                                                key1:                                                                                                                                                               ----                                                                                                                                                                value1                                                                                                                                                              key2:                                                                                                                                                               ----                                                                                                                                                                value2                                                                                                                                                              key3:                                                                                                                                                               ----                                                                                                                                                                value3                                                                                                                                                              
BinaryData                                                                                                                                                          ====                                                                                                                                                                Events:  <none>    


3. В манифесте deployment.yaml изменить спецификацию volume типа emptyDir, который монтируется в init и основной контейнер, на pvc, созданный в предыдущем
пункте


Смотрим исходное состояние пода

 % kubectl describe pod hw-deployment-5647966557-6g9qv
Name:             hw-deployment-5647966557-6g9qv
Namespace:        homework
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Thu, 18 Jul 2024 07:18:51 +0300
Labels:           app=hw
                  pod-template-hash=5647966557
Annotations:      <none>
Status:           Running
IP:               10.244.0.152
IPs:
  IP:           10.244.0.152
Controlled By:  ReplicaSet/hw-deployment-5647966557
Init Containers:
  init-container:
    Container ID:  docker://5e4e77c66e94ca7ac4308dd7b6a956ef8a686044109811147d54653d1223d1ce
    Image:         busybox:1.28
    Image ID:      docker-pullable://busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      date >> /init/index.html
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sat, 20 Jul 2024 07:12:38 +0300
      Finished:     Sat, 20 Jul 2024 07:12:38 +0300
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /init from shared-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7npj7 (ro)
Containers:
  web-server:
    Container ID:   docker://ded7c80eda40612ad8c66cb94e79bb63c84ae30b4c960420f0ac22de5d4927a8
    Image:          bakavets/kuber:v1.0
    Image ID:       docker-pullable://bakavets/kuber@sha256:fcd1325f6b0642739bb8e608ee490497c754fd86c09766b29a6a560a316cddc5
    Port:           8000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 20 Jul 2024 07:12:38 +0300
    Last State:     Terminated
      Reason:       Error
      Exit Code:    255
      Started:      Thu, 18 Jul 2024 07:18:52 +0300
      Finished:     Sat, 20 Jul 2024 07:12:27 +0300
    Ready:          True
    Restart Count:  1
    Readiness:      http-get http://:8000/index.html delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /homework from shared-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7npj7 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  shared-data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-7npj7:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              homework=true
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>



Видим:
Volumes:
  shared-data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)


Применяем изменения:

% kubectl apply -f ./deployment.yaml                 
deployment.apps/hw-deployment configured


Смотрим новое состояние

 % kubectl describe pod hw-deployment-9d49b86bf-cgczf 
Name:             hw-deployment-9d49b86bf-cgczf
Namespace:        homework
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Wed, 24 Jul 2024 05:55:46 +0300
Labels:           app=hw
                  pod-template-hash=9d49b86bf
Annotations:      <none>
Status:           Running
IP:               10.244.0.156
IPs:
  IP:           10.244.0.156
Controlled By:  ReplicaSet/hw-deployment-9d49b86bf
Init Containers:
  init-container:
    Container ID:  docker://26dab90f1d1292fb6430acf25afd9ba40986dbe9c9b58f1bda4e4124435cd569
    Image:         busybox:1.28
    Image ID:      docker-pullable://busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      date >> /init/index.html
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 24 Jul 2024 05:55:47 +0300
      Finished:     Wed, 24 Jul 2024 05:55:47 +0300
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /init from shared-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wxkpb (ro)
Containers:
  web-server:
    Container ID:   docker://926ef747f1173860ee654e2d561923749fc69db74580d1a9c4f5d65f12abad18
    Image:          bakavets/kuber:v1.0
    Image ID:       docker-pullable://bakavets/kuber@sha256:fcd1325f6b0642739bb8e608ee490497c754fd86c09766b29a6a560a316cddc5
    Port:           8000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 24 Jul 2024 05:55:48 +0300
    Ready:          True
    Restart Count:  0
    Readiness:      http-get http://:8000/index.html delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /homework from shared-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wxkpb (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  shared-data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  hw-pvc
    ReadOnly:   false
  kube-api-access-wxkpb:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              homework=true
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  38s   default-scheduler  Successfully assigned homework/hw-deployment-9d49b86bf-cgczf to minikube
  Normal  Pulled     37s   kubelet            Container image "busybox:1.28" already present on machine
  Normal  Created    37s   kubelet            Created container init-container
  Normal  Started    37s   kubelet            Started container init-container
  Normal  Pulled     36s   kubelet            Container image "bakavets/kuber:v1.0" already present on machine
  Normal  Created    36s   kubelet            Created container web-server
  Normal  Started    36s   kubelet            Started container web-server


Видим:
Volumes:
  shared-data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  hw-pvc
    ReadOnly:   false



4. В манифесте deployment.yaml добавить монтирование ранее созданного configMap как volume к основному контейнеру пода в директорию /homework/conf, так, чтобы
его содержимое можно было получить, обратившись по url /conf/file

Чтобы открывался config файл открывался по ссылке /conf/file, я добавила правила в nginx.conf, чтобы папка /homework/ считалась корневой. Для переопределения nginx.conf использую также ConfigMap

Применяем изменения:

% kubectl apply -f ./cm.yaml     
configmap/hw-configmap unchanged
configmap/nginx-config configured

% kubectl apply -f ./deployment.yaml                
deployment.apps/hw-deployment configured


% kubectl describe pod hw-deployment-bdc788797-gfjgz

Containers:
  web-server:
    ...
    Mounts:
      /etc/nginx/nginx.conf from nginx-config-volume (rw,path="nginx.conf")
      /homework from shared-data (rw)
      /homework/conf from config-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ww5vf (ro)


Volumes:
  shared-data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  hw-pvc
    ReadOnly:   false
  config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      hw-configmap
    Optional:  false
  nginx-config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      nginx-config
    Optional:  false

<<K9s-Shell>> Pod: homework/hw-deployment-bdc788797-gfjgz | Container: web-server 
root@hw-deployment-bdc788797-gfjgz:/# ls -la /homework/conf

total 12
drwxrwxrwx 3 root root 4096 Jul 24 03:04 .
drwxrwxrwx 3 root root 4096 Jul 24 03:04 ..
drwxr-xr-x 2 root root 4096 Jul 24 03:04 ..2024_07_24_03_04_56.205487400
lrwxrwxrwx 1 root root   31 Jul 24 03:04 ..data -> ..2024_07_24_03_04_56.205487400
lrwxrwxrwx 1 root root   11 Jul 24 03:04 key1 -> ..data/key1
lrwxrwxrwx 1 root root   11 Jul 24 03:04 key2 -> ..data/key2
lrwxrwxrwx 1 root root   11 Jul 24 03:04 key3 -> ..data/key3



root@hw-deployment-bdc788797-gfjgz:/# curl homework.otus/conf/key1
value1

Работает.


* Создать манифест storageClass.yaml описывающий объект типа storageClass c provisioner https://k8s.io/minikube-hostpath и reclaimPolicy Retain



% kubectl apply -f ./storageClass.yaml
storageclass.storage.k8s.io/hw-sc created

% kubectl get sc 
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
hw-sc                k8s.io/minikube-hostpath   Retain          Immediate           false                  5m16s
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  19d


 % kubectl apply  -f ./pvc.yaml                                        
persistentvolumeclaim/hw-pvc created


 % kubectl get pvc              
NAME     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
hw-pvc   Bound    pvc-2caaea00-e221-470f-830c-fdda0a6d8f6f   1Gi        RWO            hw-sc          <unset>                 19s

 % kubectl apply  -f ./deployment.yaml 
deployment.apps/hw-deployment created

# проверяем pvc на диске

% minikube ssh
docker@minikube:~$ sudo su
# cd /tmp
root@minikube:/tmp# ls
gvisor  h.1248  h.1308  hostpath-provisioner  hostpath_pv

# cd hostpath-provisioner/
root@minikube:/tmp/hostpath-provisioner# ls
homework

# cd homework/
# ls
hw-pvc

root@minikube:/tmp/hostpath-provisioner/homework/hw-pvc# ls
conf  hello.html  index.html

# Удаляем все объекты

 % kubectl delete deploy,service,configmap,pvc,ingress --all -n homework           

deployment.apps "hw-deployment" deleted
service "hw-service" deleted
configmap "hw-configmap" deleted
configmap "kube-root-ca.crt" deleted
configmap "nginx-config" deleted
persistentvolumeclaim "hw-pvc1" deleted


# смотрим pvc

root@minikube:/tmp/hostpath-provisioner/homework/hw-pvc# ls
conf  hello.html 

Файлы остались. index.html удалился при удалении пода.


