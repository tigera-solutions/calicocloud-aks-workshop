# Module 0: Creating AKS cluster

The following guide is based upon the repos from [lastcoolnameleft](https://github.com/lastcoolnameleft/kubernetes-workshop/blob/master/create-aks-cluster.md) and [Azure Kubernetes Hackfest](https://github.com/Azure/kubernetes-hackfest/tree/master/labs/create-aks-cluster#readme).

* * *

**Goal:** Create AKS cluster.

> This workshop uses AKS cluster with Linux containers. To create a Windows Server container on an AKS cluster, consider exploring [AKS documents](https://docs.microsoft.com/en-us/azure/aks/windows-container-cli). This cluster deployment utilizes Azure CLI v2.x from your local terminal or via Azure Cloud Shell. Instructions for installing Azure CLI can be found [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

If you already have an AKS cluster, make sure the network plugin is "azure", then you can skip this module and go to [module 2](/Applications/Joplin.app/Contents/Resources/modules/joining-aks-to-calico-cloud.md "../modules/joining-aks-to-calico-cloud.md")

## Prerequisite Tasks

Follow the prerequisite steps if you need to verify your Azure subscription and Service Principal otherwise proceed to step 1.

1. Ensure you are using the correct Azure subscription you want to deploy AKS to.

    ```bash
    # View subscriptions
    az account list

    # Verify selected subscription
    az account show

    # Set correct subscription (if needed)
    az account set --subscription <subscription_id>

    # Verify correct subscription is now set
    az account show
    ```

2. *[Optional]* Create Azure Service Principal for AKS resource

    >Use this step **only** if you cannot create an AKS cluster with a managed identity, otherwise skip this step. If you do use your custom service principal to create the AKS cluster, you'll have to add `--service-principal <SP_CLIENT_ID>` and `--client-secret <SP_CLIENT_SECRET>` parameter to the `az aks create ...` command below.

    - Create service principal account.

    ```bash
    az ad sp create-for-rbac --skip-assignment
    ```

    - This will return the following. **IMPORTANT** - Note this information as you'll need it to provision the AKS cluster. You'll need to use `appId` as the `--service-principal` parameter value and `password` as the `--client-secret` parameter value.

    ```bash
    # this is an example output
    “appId”: “ce493696-xxxx-xxxx-xxxx-99d6cfd09436",
    “displayName”: “jessieazurexxxx”,
    “name”: “ce493696-xxxx-xxxx-xxxx-99d6cfd09436",
    “password”: “YhutxxxxYwNxxxxQntNxxxx_VyVnxxxx”,
    “tenant”: “b0497432-xxxx-xxxx-xxxx-7d4072ca6006"
    ```

    - Set the values from above as variables **(replace `appId` and `password` with your values)**.

    >**Warning:** Several of the following steps save some variables to the `.bashrc` file. This is done so that you can get those values back if your session disconnects. You will want to clean these up once you're done with this workshop.

    **Make sure to replace the `<appId>` and `<password>` tokens below with the values from your newly created service principal object.**

    ```bash
    # Persist for Later Sessions in Case of Timeout
    APPID=<appId>
    echo export APPID=$APPID >> ~/.bashrc
    CLIENTSECRET=<password>
    echo export CLIENTSECRET=$CLIENTSECRET >> ~/.bashrc
    ```

## Steps

1. Create a unique identifier suffix for resources to be created in this lab.

    ```bash
    UNIQUE_SUFFIX=$USER$RANDOM
    # Remove Underscores and Dashes (Not Allowed in AKS and ACR Names)
    UNIQUE_SUFFIX="${UNIQUE_SUFFIX//_}"
    UNIQUE_SUFFIX="${UNIQUE_SUFFIX//-}"
    # Check Unique Suffix Value (Should be No Underscores or Dashes)
    echo $UNIQUE_SUFFIX
    # Persist for Later Sessions in Case of Timeout
    echo export UNIQUE_SUFFIX=$UNIQUE_SUFFIX >> ~/.bashrc
    ```

    **Note this value as it will be used in the next couple labs.**

2. Create an Azure Resource Group in desired region.

    ```bash
    # Set Resource Group Name using the unique suffix
    RGNAME=aks-rg-$UNIQUE_SUFFIX
    # Persist for Later Sessions in Case of Timeout
    echo export RGNAME=$RGNAME >> ~/.bashrc
    # Set Region (Location)
    LOCATION=eastus
    # Persist for Later Sessions in Case of Timeout
    echo export LOCATION=$LOCATION >> ~/.bashrc
    # Create Resource Group
    az group create -n $RGNAME -l $LOCATION
    ```

3. Create your AKS cluster in the resource group created in step 2 with 3 nodes. We will check for a recent version of Kubernetes before proceeding. You will use the Service Principal information from the prerequisite tasks.

    Use Unique CLUSTERNAME

    ```bash
    # Set AKS Cluster Name
    CLUSTERNAME=aks${UNIQUE_SUFFIX}
    # Look at AKS Cluster Name for Future Reference
    echo $CLUSTERNAME
    # Persist for Later Sessions in Case of Timeout
    echo export CLUSTERNAME=${CLUSTERNAME} >> ~/.bashrc
    ```

    Get available Kubernetes versions for the region. You will likely see more recent versions in your lab.

    ```bash
    az aks get-versions -l $LOCATION --output table
    ```

    ```text
    KubernetesVersion    Upgrades
    -------------------  ------------------------
    1.24.0(preview)      None available
    1.23.8               1.24.0(preview)
    1.23.5               1.23.8, 1.24.0(preview)
    1.22.11              1.23.5, 1.23.8
    1.22.6               1.22.11, 1.23.5, 1.23.8
    1.21.14              1.22.6, 1.22.11
    1.21.9               1.21.14, 1.22.6, 1.22.11
    ```

    For this lab we'll use 1.22.11

    ```bash
    K8SVERSION=1.22.11
    echo export K8SVERSION=$K8SVERSION >> ~/.bashrc
    ```

    >The below command can take several minutes to run as it is creating the AKS cluster.

    ```bash
    # Create AKS Cluster - it is important to set the network-plugin as azure in order to connect to Calico Cloud
    az aks create -n $CLUSTERNAME -g $RGNAME \
    --kubernetes-version $K8SVERSION \
    --enable-managed-identity \
    --node-count 3 \
    --network-plugin azure \
    --no-wait \
    --generate-ssh-keys
    ```

4. Verify your cluster status. The `ProvisioningState` should be `Succeeded`

    ```bash
    az aks list -o table -g $RGNAME
    ```

    ```bash
    Name                 Location    ResourceGroup      KubernetesVersion    ProvisioningState    Fqdn
    -------------------  ----------  -----------------  -------------------  -------------------  ----------------------------------------------------------------
    aks-calicocloud-repo    eastus      aks-rg-jessie    1.22.11               Succeeded            aks-calico-aks-rg-jessie-03cfb8-b45d6762.hcp.eastus.azmk8s.io
    ```

5. Get the Kubernetes config files for your new AKS cluster

    ```bash
    az aks get-credentials -n $CLUSTERNAME -g $RGNAME
    ```

6. Verify you have API access to your new AKS cluster

    > Note: It can take 5 minutes for your nodes to appear and be in READY state. You can run `watch kubectl get nodes` to monitor status. Otherwise, the cluster is ready when the output is similar to the following:

    ```bash
    kubectl get nodes
    ```

    ```bash
    NAME                                STATUS   ROLES   AGE   VERSION
    aks-nodepool1-36555681-vmss000000   Ready    agent   47m   v1.22.11
    aks-nodepool1-36555681-vmss000001   Ready    agent   47m   v1.22.11
    aks-nodepool1-36555681-vmss000002   Ready    agent   47m   v1.22.11

    ```

    To see more details about your cluster:

    ```bash
    kubectl cluster-info
    ```

7. Download this repo into your environment:

    ```bash
    git clone https://github.com/tigera-solutions/calicocloud-aks-workshop.git
    ```

    ```bash
    cd ./calicocloud-aks-workshop
    ```

8. *[Optional]*  Install `calicoctl` CLI to use in later labs

    a) CloudShell

    ```bash
    CALICOVERSION=$(kubectl get clusterinformations default -ojsonpath='{.spec.cnxVersion}')
    # download and configure calicoctl
    curl -o calicoctl -O -L https://downloads.tigera.io/ee/binaries/$CALICOVERSION/calicoctl

    chmod +x calicoctl
    
    # verify calicoctl is running 
    ./calicoctl version
    ```

    b) Linux
    >Tip: Consider navigating to a location that’s in your PATH. For example, /usr/local/bin/

    ```bash
    CALICOVERSION=$(kubectl get clusterinformations default -ojsonpath='{.spec.cnxVersion}')
    # download and configure calicoctl
    curl -o calicoctl -O -L https://downloads.tigera.io/ee/binaries/$CALICOVERSION/calicoctl
    chmod +x calicoctl
    
    # verify calicoctl is running 
    calicoctl version
    ```

    c) MacOS
    >Tip: Consider navigating to a location that’s in your PATH. For example, /usr/local/bin/

    ```bash
    CALICOVERSION=$(kubectl get clusterinformations default -ojsonpath='{.spec.cnxVersion}')
    # download and configure calicoctl
    curl -o calicoctl -O -L  https://downloads.tigera.io/ee/binaries/$CALICOVERSION/calicoctl-darwin-amd64

    chmod +x calicoctl
    
    # verify calicoctl is running 
    calicoctl version
    ```

    Note: If you are faced with `cannot be opened because the developer cannot be verified` error when using `calicoctl` for the first time. go to `Applicaitons` \> `System Prefences` \> `Security & Privacy` in the `General` tab at the bottom of the window click `Allow anyway`.  
    Note: If the location of calicoctl is not already in your PATH, move the file to one that is or add its location to your PATH. This will allow you to invoke it without having to prepend its location.

    d) Windows - using powershell command to download the calicoctl binary  
    >Tip: Consider runing powershell as administraor and navigating to a location that’s in your PATH. For example, C:\Windows.

    ```pwsh
    $CALICOVERSION=$(kubectl get clusterinformations default -ojsonpath='{.spec.cnxVersion}')
    Invoke-WebRequest -Uri "https://downloads.tigera.io/ee/binaries/$CALICOVERSION/calicoctl-windows-amd64.exe" -OutFile "kubectl-calico.exe"
    ```

---

## Next steps

You should now have a Kubernetes cluster running with 3 nodes. You do not see the master servers for the cluster because these are managed by Microsoft. The Control Plane services which manage the Kubernetes cluster such as scheduling, API access, configuration data store and object controllers are all provided as services to the nodes.

[Next -> Module 1](../modules/joining-aks-to-calico-cloud.md)
