# Calico cloud workshop on AKS

## Workshop objectives

The intent of this workshop is to educate any person working with AKS cluster in one way or another about Calico cloud features and how to use them. While there are many capabilities that Calico provides, this workshop focuses on a subset of those that are used most often by different types of technical users.

## Use cases

In this workshop we are going to focus on these main use cases:

- **East-West security**, leveraging zero-trust security approach.
- **Egress access controls**, using DNS policy to access external resources by their fully qualified domain names (FQDN).
- **Observability**, exploring various logs and application level metrics collected by Calico.
- **Compliance**, providing proof of security compliance.
- **Host micro-segmentation**, leveraging Calico policies to protect host ports and host based services.

## Join the Slack Channel

[Calico User Group Slack](https://slack.projectcalico.org/) is a great resource to ask any questions about Calico. If you are not a part of this Slack group yet, we highly recommend [joining it](https://slack.projectcalico.org/) to participate in discussions or ask questions. For example, you can ask questions specific to EKS and other managed Kubernetes services in the `#eks-aks-gke-iks` channel.

## Workshop prerequisites

>It is recommended to use your personal Azure account which would have full access to Azure resources. If using a corporate Azure account for the workshop, make sure to check with account administrator to provide you with sufficient permissions to create and manage AkS clusters and Load Balancer resources.

- [Calico Cloud trial account](https://www.calicocloud.io/home)
- Azure account and credentials to manage Azure resources
- Terminal or Command Line console to work with Azure resources and AKS cluster
 
- `Git`
- `netcat`

## Modules

[If you already have AKS cluster, make sure the network plugin is "azure", then you can skip module 1 & 2, and go to module 3](../modules/joining-aks-to-calico-cloud.md)

<network-plugin-azure.png>

- [Module 1: Setting up workspace environment](./modules/setting-up-work-environment.md)
- [Module 2: Creating AKS cluster](modules/creating-aks-cluster.md)
- [Module 3: Joining AKS cluster to Calico Cloud](modules/joining-eks-to-calico-cloud.md)
- [Module 4: Configuring demo applications](modules/configuring-demo-apps.md)
- [Module 5: Using security controls](modules/using-security-controls.md)
- [Module 6: Using egress access controls](modules/using-egress-access-controls.md)
- [Module 7: Using observability tools](modules/using-observability-tools.md)
- [Module 8: Using compliance reports](modules/using-compliance-reports.md)
- [Module 9: Using alerts](modules/using-alerts.md)
- [Module 10: Securing EKS hosts](modules/securing-heps.md)

## Cleanup

1. Delete application stack to clean up any `loadbalancer` services.

    ```bash
    kubectl delete -f demo/dev/app.manifests.yaml
    kubectl delete -f https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/kubernetes-manifests.yaml
    ```

2. Delete EKS cluster.

    ```bash
    eksctl delete cluster --name tigera-workshop
    ```

3. Delete EC2 Key Pair.

    ```bash
    export KEYPAIR_NAME='<set_keypair_name>'
    aws ec2 delete-key-pair --key-name $KEYPAIR_NAME
    ```

4. Delete Cloud9 instance.

    Navigate to `AWS Console` > `Services` > `Cloud9` and remove your workspace environment, e.g. `tigera-workshop`.

5. Delete IAM role created for this workshop.

    ```bash
    # use your local shell to set AWS credentials
    export AWS_ACCESS_KEY_ID="<your_accesskey_id>"
    export AWS_SECRET_ACCESS_KEY="<your_secretkey>"

    # delete IAM role
    IAM_ROLE='tigera-workshop-admin'
    ADMIN_POLICY_ARN=$(aws iam list-policies --query 'Policies[?PolicyName==`AdministratorAccess`].Arn' --output text)
    aws iam detach-role-policy --role-name $IAM_ROLE --policy-arn $ADMIN_POLICY_ARN
    # if this command fails, you can remove the role via AWS Console once you delete the Cloud9 instance
    aws iam delete-instance-profile --instance-profile-name $IAM_ROLE
    aws iam delete-role --role-name $IAM_ROLE
    ```
