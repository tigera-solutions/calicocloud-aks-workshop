# Module 10: Honeypod Threat Detection

**Goal:** Deploy honeypod resources and generate alerts when suspicous traffic is detected
---

Calico offers [Honeypod](https://docs.tigera.io/threat/honeypod/) capability which is based upon the same principles as traditional honeypots. Calico is able to detect traffic which probes the Honeypod resources which can be an indicator of compromise. Refer to the [official honeypod configuration documentation](https://docs.tigera.io/threat/honeypod/honeypods) for more details.
<br>

Note: in order to deploy honeypod resources a pull secret is required to download images from Tigera's respository. This will be supplied to you as part of this workshop 

## Steps

1. Configure honeypod namespace and Alerts for SSH detection

    ```bash
    # create dedicated namespace and RBAC for honeypods
    kubectl apply -f https://docs.tigera.io/manifests/threatdef/honeypod/common.yaml 
    # add tigera pull secret to the namespace
    # replace <pull-secrets.json> with path to your pull secret. The pull secret must be obtained from Tigera
    kubectl create secret generic tigera-pull-secret --from-file=.dockerconfigjson=<pull-secrets.json> --type=kubernetes.io/dockerconfigjson -n tigera-internal
    ```

2. Deploy sample honeypods

    ```bash
    # expose pod IP to test IP enumeration use case
    kubectl apply -f https://docs.tigera.io/manifests/threatdef/honeypod/ip-enum.yaml

    # expose nginx service that can be reached via ClusterIP or DNS
    kubectl apply -f https://docs.tigera.io/manifests/threatdef/honeypod/expose-svc.yaml 

    # expose MySQL service
    kubectl apply -f https://docs.tigera.io/manifests/threatdef/honeypod/vuln-svc.yaml 
    ```
3. Verify newly deployed pods are running

    ```bash
    kubectl get pods -n tigera-internal
    kubectl get pods -n tigera-intrusion-detection | grep -i honeypod
    ```
    >Output should resemble:
    
    ```bash
    kubectl get pods -n tigera-internal
    
    NAME                                         READY   STATUS    RESTARTS   AGE
    tigera-internal-app-ks9s4                    1/1     Running   0          7h22m
    tigera-internal-app-lfnzz                    1/1     Running   0          7h22m
    tigera-internal-app-xfl57                    1/1     Running   0          7h22m
    tigera-internal-dashboard-5779bcb9bf-h8rdl   1/1     Running   0          19h
    tigera-internal-db-58547d8655-znhkj          1/1     Running   0          19h
    ```
4. Verify honeypod alerts are deployed

    ```bash
    kubectl get globalalerts | grep -i honeypod
    ```
    >Output should resemble:

    ```bash
    kubectl get globalalerts | grep -i honeypod
    honeypod.fake.svc         2021-07-22T15:33:37Z
    honeypod.ip.enum          2021-07-21T18:34:49Z
    honeypod.network.ssh      2021-07-21T18:30:47Z
    honeypod.port.scan        2021-07-21T18:34:50Z
    honeypod.vuln.svc         2021-07-21T18:36:34Z
    ```
5. Test honeypod use cases

    - Ping exposed Honeypod IP

    ```bash
    POD_IP=$(kubectl -n tigera-internal get po --selector app=tigera-internal-app -o jsonpath='{.items[0].status.podIP}')
    kubectl -n dev exec netshoot -- ping -c5 $POD_IP
    ```
    >Output should resemble:    
    
    ```bash
    kubectl -n dev exec netshoot -- ping -c5 $POD_IP
    PING 10.240.0.74 (10.240.0.74) 56(84) bytes of data.
    64 bytes from 10.240.0.74: icmp_seq=1 ttl=63 time=0.103 ms
    64 bytes from 10.240.0.74: icmp_seq=2 ttl=63 time=0.065 ms
    64 bytes from 10.240.0.74: icmp_seq=3 ttl=63 time=0.059 ms
    64 bytes from 10.240.0.74: icmp_seq=4 ttl=63 time=0.050 ms
    64 bytes from 10.240.0.74: icmp_seq=5 ttl=63 time=0.075 ms

    --- 10.240.0.74 ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 4094ms
    rtt min/avg/max/mdev = 0.050/0.070/0.103/0.018 ms
    ```
    <br>
    
    - curl HoneyPod nginx service
    ```bash
    SVC_URL=$(kubectl -n tigera-internal get svc -l app=tigera-dashboard-internal-debug -ojsonpath='{.items[0].metadata.name}')
    SVC_PORT=$(kubectl -n tigera-internal get svc -l app=tigera-dashboard-internal-debug -ojsonpath='{.items[0].spec.ports[0].port}')
    kubectl -n dev exec netshoot -- curl -m3 -skI $SVC_URL.tigera-internal:$SVC_PORT | grep -i http
    ```
    >Output should resemble: 
    ```bash
    kubectl -n dev exec netshoot -- curl -m3 -skI $SVC_URL.tigera-internal:$SVC_PORT
    HTTP/1.1 200 OK
    Server: nginx/1.16.1
    Date: Fri, 23 Jul 2021 21:32:31 GMT
    Content-Type: text/html
    Content-Length: 112
    Last-Modified: Mon, 30 Dec 2019 17:35:18 GMT
    Connection: keep-alive
    ETag: "5e0a3556-70"
    Accept-Ranges: bytes
    ```
    <br>
    
    - Query HoneyPod MySQL service
    ```bash
    SVC_URL=$(kubectl -n tigera-internal get svc -l app=tigera-internal-backend -ojsonpath='{.items[0].metadata.name}')
    SVC_PORT=$(kubectl -n tigera-internal get svc -l app=tigera-internal-backend -ojsonpath='{.items[0].spec.ports[0].port}')
    kubectl -n dev exec netshoot -- nc -zv $SVC_URL.tigera-internal $SVC_PORT
    ```
    >Output should resemble
    ```bash
    kubectl -n dev exec netshoot -- nc -zv $SVC_URL.tigera-internal $SVC_PORT
    Connection to tigera-internal-backend.tigera-internal 3306 port [tcp/mysql] succeeded!
    ```

Head to `Alerts` view in the Enterprise Manager UI to view the related alerts. Note the alerts can take a few minutes to generate. 

<img src="../img/honeypod-threat-alert.png" alt="honeypod-threat-alert" width="100%"/>


**Congratulations on completing this Workshop!**
