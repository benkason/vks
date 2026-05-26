# Petclinic example app deployment

The Petclinic app has a Kubernetes version that is stored in Github here:
https://github.com/spring-petclinic/spring-petclinic-cloud

## Deployment steps

1. Select the cluster where you want to deploy Petclinic, and get the
    credentials from the PKS CLI:

    ```bash
    pks get-credentials <CLUSTER-NAME>
    ```

1. Clone the spring-petclinic-cloud repository:

    ```bash
    git clone https://github.com/spring-petclinic/spring-petclinic-cloud.git
    cd spring-petclinic-cloud/
    ```

1. Create the initial k8s objects for the Spring Petclinic app:

    ```bash
    kubectl apply -f k8s/init-namespace/
    kubectl apply -f k8s/init-services/
    ```

1. Log into the Wavefront UI and retrieve your token.

1. Create a secret to store your Wavefront URL and token:

    ```bash
    kubectl create secret generic wavefront -n spring-petclinic \
        --from-literal=wavefront-url=https://binz.wavefront.com \
        --from-literal=wavefront-api-token=<YOUR WAVEFRONT TOKEN>
    ```

1. Check whether your cluster has a default `StorageClass` available:

    ```bash
    kubectl get sc
    ```

    If there is already a `StorageClass` present, you will see something like
    this:

    ```
    NAME                        PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
    default-vsphere (default)   kubernetes.io/vsphere-volume   Delete          Immediate           false                  4s
    ```

1. _(optional)_ If you did not see a default `StorageClass` in the previous step,
    you must create one:

    ```bash
    kubectl apply -f - <<EOF
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: default-vsphere
      annotations:
        storageclass.kubernetes.io/is-default-class: "true"
    provisioner: kubernetes.io/vsphere-volume
    EOF
    ```

1. Use Helm to deploy the app's database:

    ```bash
    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm repo update
    helm install vets-db-mysql bitnami/mysql --namespace spring-petclinic --set auth.database=service_instance_db
    helm install visits-db-mysql bitnami/mysql --namespace spring-petclinic --set auth.database=service_instance_db
    helm install customers-db-mysql bitnami/mysql --namespace spring-petclinic --set auth.database=service_instance_db
    ```

1. Wait for the database Pods to stay Status: Running:

    ```bash
    kubectl -n spring-petclinic get pods
    ```

1. Run the helper script to deploy the Sprint petclinic app:

    ```bash
    REPOSITORY_PREFIX=docker.io/springcommunity ./scripts/deployToKubernetes.sh
    ```

1. Get the external IP of the API gateway's service:

    ```bash
    kubectl get svc -n spring-petclinic api-gateway
    ```

    You can now reach that IP from your browser using http (not https).

## (optional) Configure Ingress

If you would like to configure an Ingress to reach the Petclinic app instead of
using the LoadBalancer IP, you can apply the extra objects found in this repo:

1. Clone this repository and navigate into it:

    ```bash
    git clone git@github.com:MTNNigeria/ng-tkgi-platform.git
    cd ng-tkgi-platform/
    ```

1. Apply the Petclinic addon yamls:

   ```bash
   kubectl apply -f  k8s-manifests/petclinic-addons/

## Wavefront dashboards

You can view the metrics from the PetClinic app on these dashboards:
+ [Traces dashboard](https://binz.wavefront.com/tracing)
+ [App Operations dashboard](https://binz.wavefront.com/tracing/operation/details)
+ [Spring boot built-in dashboards](https://binz.wavefront.com/integration/springboot/content#_v01(l:()))
