# Wavefront Proxy deployment

Currently our environments are monitored by Wavefront.
Each environment (Sandbox, Preprod and Prod) has a separate cluster that runs
the Wavefront proxy, and all other clusters send their metrics to the proxy
running in that cluster, to be sent to the cloud.

![Wavefront](images/Wavefront.jpg)

## Deployment steps

These steps are adapted from here: https://docs.wavefront.com/proxies_kube_container.html

1. In the correct environment, create a dedicated cluster for your Wavefront
    Proxy:

    ```bash
    pks create-cluster <YOUR_CLUSTER_NAME> \
        --external-hostname <YOUR_CLUSTER_FQDN> \
        --plan small \
        --num-nodes 3 \
        --network-profile <YOUR_CLUSTER_NETWORK_PROFILE>
    ```

    Ask Lekan Ogunwale <lekan.ogunwale@dellteam.com> or Ben Ighagbon
    <ben.ighagbon1@dellteam.com> for the correct network profile to use in your
    environment.

1. Watch the cluster creation until it completes.

1. Obtain the cluster Master IP:
   ```bash
    pks cluster <YOUR_CLUSTER_NAME>
   ```
   Note the value of `Kubernetes Master IP(s)`

1. Add the cluster IP into your hosts file:

    ```bash
    sudo vim /etc/hosts
    # Add this:
    # <Kubernetes Master IP address>  <Cluster DNS name>
    ```

1. Get credentials and set your context to use the newly created cluster:

    ```bash
    pks get-credentials <YOUR_CLUSTER_NAME>
    kubectl config use-context <YOUR_CLUSTER_NAME>
    ```

1. Go to the [k8s-manifests/wavefront-proxy](../k8s-manifests/wavefront-proxy/)
    path in this repo.

1. Apply all of the Wavefront Kubernetes objects stored in the folder:

    ```bash
    kubectl apply -f .
    ```

1. Obtain your Wavefront API token from [here](https://binz.wavefront.com/userprofile/apiaccess)

1. Create a secret to store your Wavefront Token:

    ```bash
    kubectl -n wavefront-system create secret generic wavefront-token \
        --from-literal=WAVEFRONT_TOKEN='<YOUR_WAVEFRONT_TOKEN>'
    ```
