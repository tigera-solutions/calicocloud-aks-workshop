# Module 1: Setting up work environment

**Goal:** Set up and configure your environment to work with Azure resources.

[If you already have AKS cluster, make sure the network plugin is "azure", then you can skip this module and go to module 3](../modules/joining-aks-to-calico-cloud.md)

<network-plugin-azure.png>

## Steps

1. Ensure your environment has these tools:

    - [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
    - [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
    - [kubectl](https://kubernetes.io/docs/tasks/tools/)
    - `jq` and `netcat` utilities

    Check whether these tools already present in your environment. If not, install the missing ones.

    ```bash
    # run these commands to check whether the tools are installed in your environment
    az --version
    git --version
    kubectl version --short --client

    # install jq and netcat
    sudo yum install jq nc -y
    jq --version
    nc --version
    ```

    >For convenience consider configuring [autocompletion for kubectl](https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-bash-linux/#enable-kubectl-autocompletion).

2. Download this repo into your environment:

    ```bash
    git clone https://github.com/tigera-solutions/tigera-aks-workshop  
    ``` 

3. Login your Azure account, ensure you are using the correct Azure subscription you want to deploy AKS to.  
    >How to login your Azure account with az login, Please refer to Azure doc:
    https://docs.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest#az_login

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

4. Create Azure Service Principal to use through the labs
   ```bash
   az ad sp create-for-rbac --skip-assignment
   ```

   >This will return the following. !!!IMPORTANT!!! - Please copy this information down as you'll need it for labs going forward.

   ```bash
   "appId": "7248f250-0000-0000-0000-dbdeb8400d85",
   "displayName": "azure-cli-2017-10-15-02-20-15",
   "name": "http://azure-cli-2017-10-15-02-20-15",
   "password": "77851d2c-0000-0000-0000-cb3ebc97975a",
   "tenant": "72f988bf-0000-0000-0000-2d7cd011db47"
   ```

   Set the values from above as variables **(replace <appId> and <password> with your values)**.

    >**Warning:** Several of the following steps have you echo values to your .bashrc file. This is done so that you can get those values back if your session reconnects. You will want to remember to clean these up at the end of the training, in particular if you're running on your own, or your company's, subscription.

    DON'T MESS THIS STEP UP. REPLACE THE VALUES IN BRACKETS!!!

    ```bash
    # Persist for Later Sessions in Case of Timeout
    APPID=<appId>
    echo export APPID=$APPID >> ~/.bashrc
    CLIENTSECRET=<password>
    echo export CLIENTSECRET=$CLIENTSECRET >> ~/.bashrc
    ```


5. Create a unique identifier suffix for resources to be created in this lab.

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

   **_ Note this value as it will be used in the next couple labs. _**    


6. Create an Azure Resource Group in East US.

   ```bash
   # Set Resource Group Name using the unique suffix
   RGNAME=aks-rg-$UNIQUE_SUFFIX
   # Persist for Later Sessions in Case of Timeout
   echo export RGNAME=$RGNAME >> ~/.bashrc
   # Set Region (Location)
   LOCATION=eastus
   # Persist for Later Sessions in Case of Timeout
   echo export LOCATION=eastus >> ~/.bashrc
   # Create Resource Group
   az group create -n $RGNAME -l $LOCATION
   ```

[Next -> Module 2](../modules/creating-aks-cluster.md)

