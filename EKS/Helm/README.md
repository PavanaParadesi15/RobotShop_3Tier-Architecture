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

