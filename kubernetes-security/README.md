
# 1. B namespace homework создать service account monitoring и дать ему доступ к эндпоинту /metrics вашего кластера

% kubectl apply -f sa.yaml -f role.yaml -f rb.yaml
serviceaccount/monitoring created
clusterrole.rbac.authorization.k8s.io/metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/monitoring-metrics-reader created

% kubectl get clusterrole  --all-namespaces
NAME                                                                   CREATED AT
admin                                                                  2024-07-07T16:48:59Z
cluster-admin                                                          2024-07-07T16:48:59Z
edit                                                                   2024-07-07T16:48:59Z
ingress-nginx                                                          2024-07-17T03:43:07Z
ingress-nginx-admission                                                2024-07-17T03:43:07Z
kubeadm:get-nodes                                                      2024-07-07T16:48:59Z
metrics-reader                                                         2024-07-28T08:40:11Z
...

k9s > sa > rules

Policy(ServiceAccount:homework/monitoring)[1] 
NAME            API-GROUP       BINDING                  GET          LIST         WATCH         CREATE        PATCH         UPDATE        DELETE        DEL-LIST 
/metrics        n/a               CR:metrics-reader         ✓            ×            ×             ×             ×             ×             ×             ×       

# 2. Изменить манифест deployment из прошлых Д3 так, чтобы поды запускались под service account monitoring


% kubectl apply -f ./deployment.yaml              
deployment.apps/hw-deployment configured

Describe(homework/hw-deployment-d9448d958-5kdsq) 
Service Account:  monitoring    

<<K9s-Shell>> Pod: homework/hw-deployment-d9448d958-5kdsq | Container: web-server 
root@hw-deployment-d9448d958-5kdsq:/# cd  /var/run/secrets/kubernetes.io/serviceaccount/
root@hw-deployment-d9448d958-5kdsq:/var/run/secrets/kubernetes.io/serviceaccount# export TOKEN=$(cat ./token)

# проверяем, что работает обращение к api 
root@hw-deployment-d9448d958-5kdsq:/var/run/secrets/kubernetes.io/serviceaccount# curl --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt --header "Authorization: Bearer ${TOKEN}" -X GET https://kubernetes.default.svc/api/
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "192.168.49.2:8443"
    }
  ]
}

# проверяем, что работает обращение к /metrics
root@hw-deployment-d9448d958-5kdsq:/var/run/secrets/kubernetes.io/serviceaccount# curl --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt --header "Authorization: Bearer ${TOKEN}" -X GET https://kubernetes.default.svc/metrics/
....
workqueue_work_duration_seconds_bucket{name="priority_and_fairness_config_queue",le="1e-06"} 0
workqueue_work_duration_seconds_bucket{name="priority_and_fairness_config_queue",le="9.999999999999999e-06"} 0
workqueue_work_duration_seconds_bucket{name="priority_and_fairness_config_queue",le="9.999999999999999e-05"} 0
workqueue_work_duration_seconds_bucket{name="priority_and_fairness_config_queue",le="0.001"} 0
workqueue_work_duration_seconds_bucket{name="priority_and_fairness_config_queue",le="0.01"} 1
workqueue_work_duration_seconds_bucket{name="priority_and_fairness_config_queue",le="0.1"} 1
workqueue_work_duration_seconds_bucket{name="priority_and_fairness_config_queue",le="1"} 1
workqueue_work_duration_seconds_bucket{name="priority_and_fairness_config_queue",le="10"} 1
workqueue_work_duration_seconds_bucket{name="priority_and_fairness_config_queue",le="+Inf"} 1
workqueue_work_duration_seconds_sum{name="priority_and_fairness_config_queue"} 0.002295959
workqueue_work_duration_seconds_count{name="priority_and_fairness_config_queue"} 1



# проверяем, что не работает обращение к pods
root@hw-deployment-d9448d958-5kdsq:/var/run/secrets/kubernetes.io/serviceaccount# curl --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt --header "Authorization: Bearer ${TOKEN}" -X GET https://kubernetes.default.svc/api/v1/namespaces/homework/pods
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:homework:monitoring\" cannot list resource \"pods\" in API group \"\" in the namespace \"homework\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}

# 3. B namespace homework создать service account с именем cd и дать ему роль admin в рамках namespace homework

%  kubectl apply -f sa.yaml -f rb.yaml    
serviceaccount/monitoring unchanged
serviceaccount/cd created
secret/cd created
clusterrolebinding.rbac.authorization.k8s.io/monitoring-metrics-reader unchanged
rolebinding.rbac.authorization.k8s.io/cd-admin-binding created


% kubectl get rolebinding
NAME               ROLE                AGE
cd-admin-binding   ClusterRole/admin   18h

# 4. Создать kubeconfig для service account cd

% kubectl --kubeconfig=kubeconfig.yaml get po -n homework
NAME                            READY   STATUS    RESTARTS   AGE
hw-deployment-d9448d958-5kdsq   1/1     Running   0          42h
hw-deployment-d9448d958-wfclj   1/1     Running   0          42h
hw-deployment-d9448d958-x77kv   1/1     Running   0          42h


# 5. Сгенерировать для service account cd токен с временем действия 1 день и сохранить его в файл token

% kubectl create token cd --duration=24h > token
(base) dinakotlyar@192 kubernetes-security % cat token
eyJhbGciOiJSUzI1NiIsImtpZCI6InRYYlBwdWpoaWJtTUpzeUdOUjZQSkR6ODdRY3BpcVhoQkN4dVdZS0ZNOTgifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzIyMzkyOTc4LCJpYXQiOjE3MjIzMDY1NzgsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiNzQzNDA5YmQtZWEwZS00YmIxLWI5ZDktZGNkMGQ3YjE5YjBlIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJob21ld29yayIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJjZCIsInVpZCI6Ijc1MTQ2ZDNlLThmMGItNDRmZi1iMWYyLTk1MDg1NDAzNmExMyJ9fSwibmJmIjoxNzIyMzA2NTc4LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6aG9tZXdvcms6Y2QifQ.itXNSG4QCN8wL1mt-9CGLkc3Iws6tt3eijGPE-cFbdUxtrOThY_7J1dbxuJQfKvvCmlCyGXBs7udoHP1vTUaFt8ZkSfZuhu0E1JZEydyewMPzMp1G1OVoKJore3HrnAuMyl3WetcMTdhgyAbW2h7flwuM3k8Phip0v_PRibOskhQhLvMZp5bPgoI3b9p9lEGLf4p1a_yMS8CFeHz4SXfQQkrWOs4y1z_07PvKA0888uYkU0zMAX4_zxwotvLDjaYVLLklX6Ok87ux3NtnohPxEYZgU-M-xLmYsxoy3MRb8U7j9GAAXViclWlYrwoz1FrYpVuHCLXGXaALlByjdz0YA%               






