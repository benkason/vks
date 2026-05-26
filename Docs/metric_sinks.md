# TKGI Metric Sink configuration

In TKGI, `ClusterMetricSink` and `MetricSink` objects are used to configure
metric collection via Telegraf.
For full usage information in the public TKGI docs, see here:
https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid-Integrated-Edition/1.14/tkgi/GUID-create-sinks.html#create-clustermetricsink-and-metricsink-resources-8

Metric sinks use Telegraf, which relies on input and output plugins.
For the full list of available plugins for Telegraf, see the Telegraf Github
here: https://github.com/influxdata/telegraf/tree/1.13.4/plugins.
Each plugin's directory in that Github contains a README with configuration
options.

Input plugins that we use:
- `net_response` plugin: https://github.com/influxdata/telegraf/tree/1.13.4/plugins/inputs/net_response
- `http_response` plugin: https://github.com/influxdata/telegraf/tree/1.13.4/plugins/inputs/http_response

## Monitoring external endpoints with MetricSinks

Perform the following steps in order to monitor the uptime and latency of one or
more network endpoints.

1. Clone this repository and go into the
    [k8s-manifests/metric-sinks/](../k8s-manifests/metric-sinks/) directory.

1. Determine which cluster you want to monitor the endpoints from, and create a
    file for the `ClusterMetricSink` with the following naming convention:

    ```bash
    touch <cluster_name>-clustermetricsink.yaml
    ```

1. If this is a new file, you must do the following steps (if it already exists,
    skip):

    1. Add the following yaml config into the file:

        ```yaml
        ---
        apiVersion: pksapi.io/v1beta1
        kind: ClusterMetricSink
        metadata:
          name: default
          namespace: pks-system
        spec:
          inputs:

          outputs:
          - type: wavefront
            url: http://<YOUR LOADBALANCER INGRESS IP>:2878
            prefix: "telegraf."
        ```

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

    1. Edit your newly created file `<cluster_name>-clustermetricsink.yaml` and
        replace the placeholder `<YOUR LOADBALANCER INGRESS IP>` with the
        external IP address for the `wavefront-proxy` service.

1. For each network endpoint you would like to monitor, add the following yaml
    block into your clustermetricsink yaml file, under the `inputs`:

    ```yaml
      - type: net_response
        protocol: tcp
        address: "<ENDPOINT IP OR DNS NAME>:<ENDPOINT PORT>"
        fielddrop: ["result_type", "string_found"]
    ```

1. _(optional)_ If the endpoint is an HTTP or HTTPS endpoint, add an additional
    input config to monitor it at the application layer:

    ```yaml
      # Add this block for each endpoint
      # Set to http or https as needed
      - type: http_response
        urls:
        - "http://<ENDPOINT IP OR DNS NAME>:<ENDPOINT PORT>"
        method: GET
        follow_redirects: true
    ```

1. Login into the cluster where you want to create this metricsink:

    ```bash
    pks get-credentials <CLUSTER-NAME>
    ```

1. Run the following commands to apply the yaml file:

    ```bash
    kubectl apply -f <FILE-NAME>
    ```
