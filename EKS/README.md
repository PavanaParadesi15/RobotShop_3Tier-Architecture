# Prerequisites

* Launch a EC2 instance to install entire setup

## Install Kubectl
```
sudo apt update
sudo apt install curl -y
sudo curl -LO "https://dl.k8s.io/release/v1.28.4/bin/linux/amd64/kubectl"
sudo chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```
## Install Eksctl 
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
## Install AWSCLI
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
aws --version
```
## Configure AWS
```
aws configure
```
Give Access key and secret access key details

# Install EKS Cluster
```
eksctl create cluster --name demo-cluster-three-tier-1 --region us-east-1
```
Update EKS Cluster from one version to next version
```
aws eks update-cluster-version --name demo-cluster-three-tier-1 --kubernetes-version 1.31 --region us-east-1
```

To check the status of the update
```
aws eks describe-cluster --name demo-cluster-three-tier-1 --region us-east-1 --query 'cluster.status'
```
### Delete EKS Cluster
```
eksctl delete cluster --name demo-cluster-three-tier-1 --region us-east-1
```
## Configure IAM OIDC provider
* IAM OIDC provider is useful for EKS to interact with other AWS services like EKS.
* By default K8s creates service accounts for each pod in EKS cluster, we can integrate K8S service accounts with AWS IAM roles/policies.
* When the service acc of each pod integrated with IAM roles/policies, that particular pod can talk to other AWS services.
* When using stateful sets in EKS (in this project Redis is the stateful set).
* A StatefulSet is a way to manage applications that need to store and retrieve data, like databases or file systems. It's a Kubernetes resource that ensures each pod in the set has a unique identity and maintains its state, even if the pod is restarted or moved.
* Redis has to communicate with EKS volumes as persistant volumes is used through EBS. So service accounts of the EKS resources are integrated with IAM , so resources can communicate with other AWS resources like EBS. 

### Commands to configure IAM OIDC provider
```
export cluster_name=<CLUSTER-NAME>
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5) 
```

### Check if there is an IAM OIDC provider configured already
```
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```
If there is nothing created, create it using below command
```
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
OIDC configuration is completed


# Setup ALB - Application Load Balancer
* ALB is required to expose the application to external world. Expose it using ingress and ingress controller. 
* I created  ingress with deployment and service .
* There are 2 ways to expose application to external world in K8s,
  1. Loadbalancer type service      2. Use Ingress based service. Create ingress for the service

### Download IAM policy
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```
### Create IAM Policy
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

### Create IAM Role
```
eksctl create iamserviceaccount \
  --cluster=<your-ekscluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
* ALB controller is created in kube-system namespace, because, ALB controleer watches all the name spaces. Most of the ingress controllers are deployed in kube-system namespace.
* Above commands creates 'service account' for ALB controller. Once it is done, add the HELM chart for ALB controller, install ALB controller using Helm

## Deploy ALB controller

### Install Helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
### Add helm repo
```
helm repo add eks https://aws.github.io/eks-charts
```
Update the repo
```
helm repo update eks
```
### Install Helm for ALB controller
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=<your-cluster-name> --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=<region> --set vpcId=<your-vpc-id>
```

### Verify that the deployments (ALB controller pods) are running.
```
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl get pods -n kube-system             // shows all the pods running
```

# EBS CSI Plugin configuration
* When we are using Redis, which is created as stateful set, as Redis is used as in-memory datastore, Redis requires persistant volume like EBS orEFS.
* In this project we are using EBS volume as persistant volume. When Redis is created there are 2 components 
1. Persistant volume claim (PVC)
2. Storage class 

We need to add EBS CSI plugin to EKS, so that whenever PVC is created, EBS volume is automatically created and attached to Redis stateful set

The Amazon EBS CSI plugin requires IAM permissions to make calls to AWS APIs on your behalf.

Create an IAM role and attach a policy. AWS maintains an AWS managed policy or you can create your own custom policy. You can create an IAM role and attach the AWS managed policy with the following command. Replace my-cluster with the name of your cluster. The command deploys an AWS CloudFormation stack that creates an IAM role and attaches the IAM policy to it.

```
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster <YOUR-CLUSTER-NAME> \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve --region us-east-1
```
Once IAM service account is created, Create a addon
```
eksctl create addon --name aws-ebs-csi-driver --cluster <YOUR-CLUSTER-NAME> --service-account-role-arn arn:aws:iam::<AWS-ACCOUNT-ID>:role/AmazonEKS_EBS_CSI_DriverRole --force
```


Next deploy the complete project as HELM Chart









