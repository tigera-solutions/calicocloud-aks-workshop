# Module 7: Packet Capture

**Goal:** Configure packet capture for specific pods and review captured payload.

Packet captures are Kubernetes Custom Resources and thus native Kubernetes RBAC can be used to control which users/groups can run and access Packet Captures; this may be useful if Compliance or Governance policies mandate strict controls on running Packet Captures for specific workloads. This demo is simplified without RBAC but further details can be found [here](https://docs.tigera.io/v3.9/visibility/packetcapture).

## Steps

1. Initial packet capture job from manager UI. 

   ![packet capture](../img/packet-capture-ui.png)


2. Schedule the packet capture job with specific port.

   ![test packet capture](../img/test-packet-capture.png)


3. You will see the job scheduled in service graph.


   ![schedule packet capture](../img/schedule-packet-capture.png)


4. Download the pcap file once the job is `Capturing` or `Finished`. 
   
   ![download packet capture](../img/download-packet-capture.png)
   


   

2. Configure packet capture.

    Navigate to `demo/80-packet-capture` and review YAML manifests that represent packet capture definition. Each packet capture is configured by deploying a `PacketCapture` resource that targets endpoints using `selector` and `labels`.

    Deploy packet capture definition to capture packets for `default/frontend` pods.

    test-packet-capture.png

    ```bash
    kubectl apply -f demo/80-packet-capture/packet-capture.yaml
    ```

    >Once the `PacketCapture` resource is deployed, Calico starts capturing packets for all endpoints configured in the `selector` field.


3. Fetch and review captured payload.

    >The captured `*.pcap` files are stored on the hosts where pods are running at the time the `PacketCapture` resource is active.

    Retrieve captured `*.pcap` files and review the content.

    ```bash
    # get pcap files
    calicoctl captured-packets copy packet-capture-frontend --namespace default

    ls frontend-*
    # view *.pcap content
    tcpdump -Xr frontend-XXXXXX.pcap
    ```
    > You will need to install the tcpdump binary to follow the instructions above. If you are using `Cloud Shell` you can download the pcap file to your local terminal and use tcpdump or any other application to inspect the file. 

    ```bash
    download frontend-XXXXXX.pcap
    ```

4. Stop packet capture

    Stop packet capture by removing the `PacketCapture` resource.

    ```bash
    kubectl delete -f demo/80-packet-capture/packet-capture.yaml
    ```

[Next -> Module 8](../modules/using-compliance-reports.md)
