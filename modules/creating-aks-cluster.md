# Module 1: Creating AKS cluster

**Goal:** Create AKS cluster.

>This workshop uses AKS cluster with Linux containers. To create a Windows Server container on an AKS cluster, consider exploring [AKS documents](https://docs.microsoft.com/en-us/azure/aks/windows-container-cli).

[If you already have AKS cluster, make sure the network plugin is "azure", then you can skip this module and go to module 2](../modules/joining-aks-to-calico-cloud.md)

<img src="img/network-plugin-azure.png" alt="network plugin" width="100%"/>

## Steps

1. Create your AKS cluster in the resource group created in module 2 with 3 nodes. We will check for a recent version of kubnernetes before proceeding. We are also including the monitoring add-on for Azure Container Insights. You will use the Service Principal information from step 5.

   Use Unique CLUSTERNAME

   ```bash
   # Set AKS Cluster Name
   CLUSTERNAME=aks${UNIQUE_SUFFIX}
   # Look at AKS Cluster Name for Future Reference
   echo $CLUSTERNAME
   # Persist for Later Sessions in Case of Timeout
   echo export CLUSTERNAME=aks${UNIQUE_SUFFIX} >> ~/.bashrc
   ```

   Get available kubernetes versions for the region. You will likely see more recent versions in your lab.

   ```bash
   az aks get-versions -l $LOCATION --output table
   KubernetesVersion    Upgrades
   -------------------  ------------------------
   1.21.1(preview)      None available
   1.20.7               1.21.1(preview)
   1.20.5               1.20.7, 1.21.1(preview)
   1.19.11              1.20.5, 1.20.7
   1.19.9               1.19.11, 1.20.5, 1.20.7
   1.18.19              1.19.9, 1.19.11
   1.18.17              1.18.19, 1.19.9, 1.19.11
   ```

   For this lab we'll use 1.20.7

   ```bash
   K8SVERSION=1.20.7
   ```

   > The below command can take 10-20 minutes to run as it is creating the AKS cluster. Please be PATIENT and grab a coffee...

   ```bash
   # Create AKS Cluster
   az aks create -n $CLUSTERNAME -g $RGNAME \
   --kubernetes-version $K8SVERSION \
   --service-principal $APPID \
   --client-secret $CLIENTSECRET \
   --generate-ssh-keys -l $LOCATION \
   --node-count 3 \
   --network-plugin azure \
   --no-wait
   
   ```
2. Verify your cluster status. The `ProvisioningState` should be `Succeeded`

    ```bash
    az aks list -o table
    ```

    ```bash
    Name           Location    ResourceGroup      KubernetesVersion    ProvisioningState    Fqdn
    -------------  ----------  -----------------  -------------------  -------------------  -----------------------------------------------------------------
    aksjessie2081  eastus      aks-rg-jessie2081  1.20.7               Succeeded             aksjessie2-aks-rg-jessie208-03cfb8-9713ae4f.hcp.eastus.azmk8s.io

    ```
3. Get the Kubernetes config files for your new AKS cluster

    ```bash
    az aks get-credentials -n $CLUSTERNAME -g $RGNAME
    ```
4. Verify you have API access to your new AKS cluster

    > Note: It can take 5 minutes for your nodes to appear and be in READY state. You can run `watch kubectl get nodes` to monitor status.

    ```bash
    kubectl get nodes
    ```

    ```bash
    NAME                                STATUS   ROLES   AGE    VERSION
    aks-nodepool1-29374799-vmss000000   Ready    agent   118s   v1.20.7
    aks-nodepool1-29374799-vmss000001   Ready    agent   2m3s   v1.20.7
    aks-nodepool1-29374799-vmss000002   Ready    agent   2m     v1.20.7
    ```

    To see more details about your cluster:

    ```bash
    kubectl cluster-info
    ```

    ```bash

    Kubernetes control plane is running at https://aksjessie2-aks-rg-jessie208-03cfb8-9713ae4f.hcp.eastus.azmk8s.io:443
    CoreDNS is running at https://aksjessie2-aks-rg-jessie208-03cfb8-9713ae4f.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
    Metrics-server is running at https://aksjessie2-aks-rg-jessie208-03cfb8-9713ae4f.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

    ```

    You should now have a Kubernetes cluster running with 3 nodes. You do not see the master servers for the cluster because these are managed by Microsoft. The Control Plane services which manage the Kubernetes cluster such as scheduling, API access, configuration data store and object controllers are all provided as services to the nodes.


[Next -> Module 2](../modules/joining-aks-to-calico-cloud.md)






1. Enable network mode using the aks-preview extension.

    ```bash
    az extension add --name aks-preview
    az feature register -n AKSNetworkModePreview --namespace Microsoft.ContainerService
    az provider register -n Microsoft.ContainerService
    ```

   
2. Create AKS cluse with azure network plugin value:( In order to run Calico CNI, we will need specify the network plugin azure)

    ```bash
    az aks create --resource-group WorkshopGroup --name myAKSCluster --node-count 3 --enable-addons monitoring --generate-ssh-keys ----network-plugin azure
    ```
    > Turn off the preview addon once finish. You can save the preview as file for future deployment.
    ```bash
    az extension remove --name aks-preview 
    ```

3. Configure kubectl to connect to your Kubernetes cluster using the az aks get-credentials command. The following command will download credentials, merge your aks cluster to default kubeconfig location and configures the Kubernetes CLI to use them.
   ```bash
   az aks get-credentials --resource-group WorkshopResourceGroup --name myAKSCluster
   ```
   > You can also specify a different location for your Kubernetes configuration file using --file.


4. View AKS cluster.

    Once cluster is created you can view the dashboard of your kubernetes cluster in a web browser using`az aks`:

    ```bash
    az aks browse --name MyAKSCluster --resource-group WorkshopResourceGroup
    ```

6. Test access to AKS cluster with `kubectl`

    Once the AKS cluster is provisioned, the `kubeconfig` file would be placed into `~/.kube/config` path. The `kubectl` CLI looks for `kubeconfig` at `~/.kube/config` path or into `KUBECONFIG` env var.

    ```bash
    # verify kubeconfig file path
    ls ~/.kube/config
    # test cluster connection
    kubectl get nodes
    # view your merged kubeconfig settings. 
    kubectl config view
    ```

[Next -> Module 3](../modules/joining-aks-to-calico-cloud.md)
