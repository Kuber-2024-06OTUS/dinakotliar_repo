# 1. Создать helm-chart позволяющий деплоить приложение, которое у вас получилось при выполнении ДЗ 1-5

% helm dependency build hw-chart
Getting updates for unmanaged Helm repositories...
...Successfully got an update from the "https://charts.bitnami.com/bitnami" chart repository
Error: can't get a valid version for repositories redis. Try changing the version constraint in Chart.yaml
(base) dinakotlyar@192 kubernetes-templating % 
(base) dinakotlyar@192 kubernetes-templating % 
(base) dinakotlyar@192 kubernetes-templating % 
(base) dinakotlyar@192 kubernetes-templating % helm dependency build hw-chart
Getting updates for unmanaged Helm repositories...
...Successfully got an update from the "https://charts.bitnami.com/bitnami" chart repository
Saving 1 charts
Downloading redis from repo https://charts.bitnami.com/bitnami
Deleting outdated charts


% helm install app hw-chart     
NAME: app
LAST DEPLOYED: Sat Aug  3 07:47:42 2024
NAMESPACE: homework
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing hw-chart.

Your release is named app.

To learn more about the release, try:

  $ helm status app
  $ helm get all app

Application URL:
  http://homework.otus/

Get service ip:
    export SERVICE_IP=$(kubectl get service/app-service -o jsonpath="{.spec.clusterIP}")
    echo http://$SERVICE_IP


% helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
app     homework        1               2024-08-03 05:16:11.708942 +0300 MSK    deployed        hw-chart-0.1.0  1.0.0      


# Установить kafka из bitnami helm-чарта

% helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
% helm repo update

Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈

% kubectl create namespace prod
namespace/prod created

% kubens prod
Context "minikube" modified.
Active namespace is "prod".

# применила параметры из кастомного values файла

 % helm install kafka bitnami/kafka --namespace prod -f kafka_values_prod.yaml
NAME: kafka
LAST DEPLOYED: Sat Aug  3 10:22:19 2024
NAMESPACE: prod
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: kafka
CHART VERSION: 29.3.14
APP VERSION: 3.7.1

** Please be patient while the chart is being deployed **

Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    kafka.prod.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    kafka-controller-0.kafka-controller-headless.prod.svc.cluster.local:9092
    kafka-controller-1.kafka-controller-headless.prod.svc.cluster.local:9092
    kafka-controller-2.kafka-controller-headless.prod.svc.cluster.local:9092

The CLIENT listener for Kafka client connections from within your cluster have been configured with the following security settings:
    - SASL authentication

To connect a client to your Kafka, you need to create the 'client.properties' configuration files with the content below:

security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
    username="user1" \
    password="$(kubectl get secret kafka-user-passwords --namespace prod -o jsonpath='{.data.client-passwords}' | base64 -d | cut -d , -f 1)";

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run kafka-client --restart='Never' --image docker.io/bitnami/kafka:3.5.2 --namespace prod --command -- sleep infinity
    kubectl cp --namespace prod /path/to/client.properties kafka-client:/tmp/client.properties
    kubectl exec --tty -i kafka-client --namespace prod -- bash

    PRODUCER:
        kafka-console-producer.sh \
            --producer.config /tmp/client.properties \
            --broker-list kafka-controller-0.kafka-controller-headless.prod.svc.cluster.local:9092,kafka-controller-1.kafka-controller-headless.prod.svc.cluster.local:9092,kafka-controller-2.kafka-controller-headless.prod.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \
            --consumer.config /tmp/client.properties \
            --bootstrap-server kafka.prod.svc.cluster.local:9092 \
            --topic test \
            --from-beginning
WARNING: Rolling tag detected (bitnami/kafka:3.5.2), please note that it is strongly recommended to avoid using rolling tags in a production environment.
+info https://docs.vmware.com/en/VMware-Tanzu-Application-Catalog/services/tutorials/GUID-understand-rolling-tags-containers-index.html

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - controller.resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

⚠ SECURITY WARNING: Original containers have been substituted. This Helm chart was designed, tested, and validated on multiple platforms using a specific set of Bitnami and Tanzu Application Catalog containers. Substituting other containers is likely to cause degraded security and performance, broken chart features, and missing environment variables.

Substituted images detected:
  - docker.io/bitnami/kafka:3.5.2


# сделала через сценарий helmfile

% brew install helmfile
% helmfile sync

Adding repo bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

Upgrading release=kafka-prod, chart=bitnami/kafka, namespace=prod
Upgrading release=kafka-dev, chart=bitnami/kafka, namespace=dev
Release "kafka-prod" does not exist. Installing it now.
NAME: kafka-prod
LAST DEPLOYED: Sat Aug  3 11:28:54 2024
NAMESPACE: prod
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: kafka
CHART VERSION: 22.1.1
APP VERSION: 3.4.0

** Please be patient while the chart is being deployed **

Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    kafka-prod.prod.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    kafka-prod-0.kafka-prod-headless.prod.svc.cluster.local:9092
    kafka-prod-1.kafka-prod-headless.prod.svc.cluster.local:9092
    kafka-prod-2.kafka-prod-headless.prod.svc.cluster.local:9092
    kafka-prod-3.kafka-prod-headless.prod.svc.cluster.local:9092
    kafka-prod-4.kafka-prod-headless.prod.svc.cluster.local:9092

You need to configure your Kafka client to access using SASL authentication. To do so, you need to create the 'kafka_jaas.conf' and 'client.properties' configuration files with the content below:

    - kafka_jaas.conf:

KafkaClient {
org.apache.kafka.common.security.scram.ScramLoginModule required
username="user"
password="$(kubectl get secret kafka-prod-jaas --namespace prod -o jsonpath='{.data.client-passwords}' | base64 -d | cut -d , -f 1)";
};

    - client.properties:

security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run kafka-prod-client --restart='Never' --image docker.io/bitnami/kafka:3.5.2 --namespace prod --command -- sleep infinity
    kubectl cp --namespace prod /path/to/client.properties kafka-prod-client:/tmp/client.properties
    kubectl cp --namespace prod /path/to/kafka_jaas.conf kafka-prod-client:/tmp/kafka_jaas.conf
    kubectl exec --tty -i kafka-prod-client --namespace prod -- bash
    export KAFKA_OPTS="-Djava.security.auth.login.config=/tmp/kafka_jaas.conf"

    PRODUCER:
        kafka-console-producer.sh \
            --producer.config /tmp/client.properties \
            --broker-list kafka-prod-0.kafka-prod-headless.prod.svc.cluster.local:9092,kafka-prod-1.kafka-prod-headless.prod.svc.cluster.local:9092,kafka-prod-2.kafka-prod-headless.prod.svc.cluster.local:9092,kafka-prod-3.kafka-prod-headless.prod.svc.cluster.local:9092,kafka-prod-4.kafka-prod-headless.prod.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \
            --consumer.config /tmp/client.properties \
            --bootstrap-server kafka-prod.prod.svc.cluster.local:9092 \
            --topic test \
            --from-beginning
WARNING: Rolling tag detected (bitnami/kafka:3.5.2), please note that it is strongly recommended to avoid using rolling tags in a production environment.
+info https://docs.bitnami.com/containers/how-to/understand-rolling-tags-containers/

Listing releases matching ^kafka-prod$
kafka-prod      prod            1               2024-08-03 11:28:54.938147 +0300 MSK    deployed        kafka-22.1.1    3.4.0      

Release "kafka-dev" does not exist. Installing it now.
NAME: kafka-dev
LAST DEPLOYED: Sat Aug  3 11:28:54 2024
NAMESPACE: dev
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: kafka
CHART VERSION: 29.3.14
APP VERSION: 3.7.1

** Please be patient while the chart is being deployed **

Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    kafka-dev.dev.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    kafka-dev-controller-0.kafka-dev-controller-headless.dev.svc.cluster.local:9092
    kafka-dev-controller-1.kafka-dev-controller-headless.dev.svc.cluster.local:9092
    kafka-dev-controller-2.kafka-dev-controller-headless.dev.svc.cluster.local:9092

The CLIENT listener for Kafka client connections from within your cluster have been configured with the following security settings:
    - SASL authentication

To connect a client to your Kafka, you need to create the 'client.properties' configuration files with the content below:

security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
    username="user1" \
    password="$(kubectl get secret kafka-dev-user-passwords --namespace dev -o jsonpath='{.data.client-passwords}' | base64 -d | cut -d , -f 1)";

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run kafka-dev-client --restart='Never' --image docker.io/bitnami/kafka:3.7.1-debian-12-r4 --namespace dev --command -- sleep infinity
    kubectl cp --namespace dev /path/to/client.properties kafka-dev-client:/tmp/client.properties
    kubectl exec --tty -i kafka-dev-client --namespace dev -- bash

    PRODUCER:
        kafka-console-producer.sh \
            --producer.config /tmp/client.properties \
            --broker-list kafka-dev-controller-0.kafka-dev-controller-headless.dev.svc.cluster.local:9092,kafka-dev-controller-1.kafka-dev-controller-headless.dev.svc.cluster.local:9092,kafka-dev-controller-2.kafka-dev-controller-headless.dev.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \
            --consumer.config /tmp/client.properties \
            --bootstrap-server kafka-dev.dev.svc.cluster.local:9092 \
            --topic test \
            --from-beginning

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - controller.resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

Listing releases matching ^kafka-dev$
kafka-dev       dev             1               2024-08-03 11:28:54.938166 +0300 MSK    deployed        kafka-29.3.14   3.7.1      


UPDATED RELEASES:
NAME         NAMESPACE   CHART           VERSION   DURATION
kafka-prod   prod        bitnami/kafka   22.1.1          2s
kafka-dev    dev         bitnami/kafka   29.3.14         2s


 % helm list -n prod   
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
kafka-prod      prod            1               2024-08-03 11:28:54.938147 +0300 MSK    deployed        kafka-22.1.1    3.4.0     


% helm list -n dev 
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
kafka-dev       dev             1               2024-08-03 11:28:54.938166 +0300 MSK    deployed        kafka-29.3.14   3.7.1      

