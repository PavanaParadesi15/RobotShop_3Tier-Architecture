# Helm Chart
## Install Helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
### Create a namespace
```
kubectl create ns robot-shop
helm install robot-shop --namespace robot-shop .                    //  helm chart deploys the component
``` 

* IN the Helm chart there are 2 important components 
1. Chart.yaml               2. Values.yaml
* In the templates folder, there are all the k8s manifests/resources and managing values through values.yaml
* Advantage of using helm charts is it makes deployment of K8s resources easy and manageable. 

### Creating Ingress resource to expose the application
* Creating Ingress for the service called Web and service port is 8080 for Web and exposing it using ALB controller. 

```
kubectl apply -f ingress.yaml
kubectl get ingress -n robot-shop
```

Ingress is assigned with LB DNS name. Application can be accessed with this DNS name

![image](https://github.com/user-attachments/assets/7bf52237-7c6d-4b87-a9ea-4408fb48512d)




## Kubectl commands

```
kubectl get namespaces
kubectl create ns robot-shop
kubectl delete namespace robot-shop
kubectl get pods -n robot-shop
kubectl get svc -n robot-shop
kubectl get deployments -n robot-shop
kubectl get ingress -n robot-shop

```

Verify that the deployments are running.

```
kubectl get deployment -n kube-system aws-load-balancer-controller
```

# ERROR

We might face the issue, unable to see the loadbalancer address while giving "**kubectl get ingress -n robot-shop**" at the end. To avoid this the "**AWSLoadBalancerControllerIAMPolicy**"  should have the required permissions for  "**elasticloadbalancing:DescribeListenerAttributes**".


## Run the following command to retrieve the policy details and look for "**elasticloadbalancing:DescribeListenerAttributes**" in the policy document.
```
aws iam get-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --version-id $(aws iam get-policy --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy --query 'Policy.DefaultVersionId' --output text)
```

If the required permission is missing, update the policy to include it

## Download the current policy
```
aws iam get-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --version-id $(aws iam get-policy --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy --query 'Policy.DefaultVersionId' --output text) \
    --query 'PolicyVersion.Document' --output json > policy.json
```
## Edit policy.json to add the missing permissions
```
{
  "Effect": "Allow",
  "Action": "elasticloadbalancing:DescribeListenerAttributes",
  "Resource": "*"
}
```
## Create a new policy version
```
aws iam create-policy-version \
    --policy-arn arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://policy.json \
    --set-as-default
```






