# Wavefront collector deployment

Currently our environments are monitored by Wavefront.
Each environment (Sandbox, Preprod and Prod) has a separate cluster that runs
the Wavefront proxy. Clusters can send metrics to the proxy via the Wavefront collector.


![Wavefront](images/Wavefront.jpg)

## Deployment steps

1. From PKS jump box, run the following command to list the clusters:
    ```bash
    pks clusters
    ```
    Note the cluster with name begining with `wfproxy` e.g.: `wfproxysandbox`
1. Login to the cluster:
    ```bash
    pks get-credentials <PROXY CLUSTER NAME>
    ```
1. Switch context to the cluster where the Wavefront proxy is deployed:
    ```bash
    kubectl config use-context <PROXY CLUSTER NAME>
    ```
1. Note the external IP address for the `wavefront-proxy` service
    ```bash
    kubectl get services -n wavefront-system
    ```
1. Go to the folder [k8s-manifests/wavefront-collector](../k8s-manifests/wavefront-collector/)
1. Edit the file named `4-collector-config.yaml`
    1. Replace the placeholder `<YOUR CLUSTER NAME>` with the name of the cluster
    you are deploying the Wavefront collector into.
    1. Replace the placeholder `<YOUR LOADBALANCER INGRESS IP>` with the external
    IP address for the `wavefront-proxy` service.
1. Switch context to the cluster you wish to deploy Wavefront collector into:
    ```bash
    kubectl config use-context <CLUSTER NAME>
    ```
1. Deploy the Wavefront collector:
    ```bash
    kubectl apply -f .
    ```
1. Revert the changes you made to `4-collector-config.yaml`
1. You should now see metrics from your cluster in the Wavefront UI.
