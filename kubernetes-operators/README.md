# Создать манифест объекта CustomResourceDefinition

% kubectl apply -f crd.yaml               

customresourcedefinition.apiextensions.k8s.io/mysqls.otus.homework created

% kubectl apply -f roles.yaml     
namespace/hwmysql created
serviceaccount/mysql-operator-sa created
clusterrole.rbac.authorization.k8s.io/mysql-operator-role created
clusterrolebinding.rbac.authorization.k8s.io/mysql-operator-binding created

% kubectl apply -f deployment.yaml 
deployment.apps/mysql-operator created

% kubectl apply -f cr.yaml        
mysql.otus.homework/hw-mysql created

% kubens hwmysql
Context "minikube" modified.
Active namespace is "hwmysql"


% kubectl get crd mysqls.otus.homework

NAME                   CREATED AT
mysqls.otus.homework   2024-08-03T15:37:45Z

% kubectl get pods                      
NAME                              READY   STATUS    RESTARTS   AGE
mysql-operator-6cc465b89b-xfvt5   1/1     Running   0          8m15s


% kubectl get mysql hw-mysql
NAME       AGE
hw-mysql   8m52s


% kubectl get deployments

NAME             READY   UP-TO-DATE   AVAILABLE   AGE
mysql-operator   1/1     1            1           9m38s

% kubectl get svc

No resources found in hwmysql namespace.

% kubectl get pv

NAME               CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
hw-mysql-pv        1Gi        RWO            Retain           Released   default/hw-mysql-pvc  standard       <unset>                          8m52s


% kubectl get pvc

No resources found in hwmysql namespace.




% kubectl delete mysql hw-mysql

mysql.otus.homework "hw-mysql" deleted


# поды остались
% kubectl get pods   
NAME                              READY   STATUS    RESTARTS   AGE
mysql-operator-6cc465b89b-xfvt5   1/1     Running   0          2d23h

% kubectl get deployments
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
mysql-operator   1/1     1            1           2d23h


# CR нет
% kubectl get mysqls
No resources found in hwmysql namespace.

# PV нет
% kubectl get pv hw-mysql-pv 
Error from server (NotFound): persistentvolumes "hw-mysql-pv" not found