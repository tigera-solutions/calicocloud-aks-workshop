# Module 5: Layer 7 Logging and Visibility

**Goal:** Enable Layer 7 visibility for Pod traffic.
---

Calico Cloud can be enabled for Layer 7 application visibility which captures the HTTP calls applications are making. Application visibility does not require a service mesh but does utilise envoy for capturing logs. Envoy is deployed as part of an L7 Log Collector DaemonSet per Kubernetes node - this requires less resources than a sidecar per pod. For more info please review the [documentation](https://docs.tigera.io/v3.9/visibility/elastic/l7/configure).

## Steps

1.  Configure Felix for log data collection

    >Enable the Policy Sync API in Felix - we configure this cluster-wide

    ```bash
    kubectl patch felixconfiguration default --type='merge' -p '{"spec":{"policySyncPathPrefix":"/var/run/nodeagent"}}'
    ```

2.  Prepare scripts for deploying L7 Log Collector DaemonSet

    ```bash
    DOCS_LOCATION=${DOCS_LOCATION:="https://docs.tigera.io"}

    #Download manifest file for L7 log collector daemonset
    curl ${DOCS_LOCATION}/v3.9/manifests/l7/daemonset/l7-collector-daemonset.yaml -O

    #Download and install Envoy Config
    curl ${DOCS_LOCATION}/v3.9/manifests/l7/daemonset/envoy-config.yaml -O
    ```
3.  Create the Envoy config in `calico-system` namespace
    ```bash
    kubectl create configmap envoy-config -n calico-system --from-file=envoy-config.yaml
    ```
4.  Apply the L7 Log Collector DaemonSet manifest and patch Felix with AKS specific parameters
    ```bash
    kubectl apply -f l7-collector-daemonset.yaml
    kubectl patch installation default --type=merge -p '{"spec": {"kubernetesProvider": "AKS"}}'
    kubectl patch installation default --type=merge -p '{"spec": {"flexVolumePath": "/etc/kubernetes/volumeplugins/"}}'
    ```
5.  Enable L7 log collection daemonset mode in Felix
    ```bash
    kubectl patch felixconfiguration default --type='merge' -p '{"spec":{"tproxyMode":"Enabled"}}'
    ```
    >If successfully deployed an `l7-log-collector` pod will be deployed on each node. To verify:
    ```bash
    kubectl get pod -n calico-system
    ```
    >Output will look similar to:
    ```
    NAME                                       READY   STATUS    RESTARTS   AGE
    calico-kube-controllers-6b4dccd6c5-579s8   1/1     Running   0          120m
    calico-node-b26qh                          1/1     Running   0          120m
    calico-node-pl646                          1/1     Running   0          2m2s
    calico-node-rmx2q                          1/1     Running   0          120m
    calico-typha-6f7f966d4-28n9j               1/1     Running   0          122m
    calico-typha-6f7f966d4-8nx5f               1/1     Running   0          2m1s
    calico-typha-6f7f966d4-g7b69               1/1     Running   0          122m
    l7-log-collector-627qf                     2/2     Running   0          91s
    l7-log-collector-6b6cx                     2/2     Running   0          3m52s
    l7-log-collector-jxzjq                     2/2     Running   0          15m
    ```

6.  Annotate the Boutiqueshop Services

    ```bash
    kubectl annotate svc -n default adservice projectcalico.org/l7-logging=true
    kubectl annotate svc -n default cartservice projectcalico.org/l7-logging=true
    kubectl annotate svc -n default checkoutservice projectcalico.org/l7-logging=true
    kubectl annotate svc -n default currencyservice projectcalico.org/l7-logging=true
    kubectl annotate svc -n default emailservice projectcalico.org/l7-logging=true
    kubectl annotate svc -n default frontend projectcalico.org/l7-logging=true
    kubectl annotate svc -n default paymentservice projectcalico.org/l7-logging=true
    kubectl annotate svc -n default productcatalogservice projectcalico.org/l7-logging=true
    kubectl annotate svc -n default recommendationservice projectcalico.org/l7-logging=true
    kubectl annotate svc -n default redis-cart projectcalico.org/l7-logging=true
    kubectl annotate svc -n default shippingservice projectcalico.org/l7-logging=true
    ```
    
    >L7 flow logs will require a few minutes to generate.

7.  Review L7 logs

    The HTTP logs can be reviewed from `Service Graph` and then clicking the `HTTP` tab. Details of each flow can be reviewed by drilling down into the flow record

    <img src="../img/service-graph-l7.png" alt="Service Graph L7" width="100%"/>

[Next -> Module 6](../modules/using-observability-tools.md)




