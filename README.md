# super-barnacle.ci-cd
GitOps for super-barnacle project<br/><br/>
This Repo contains Helm Chart for managing the Kubernetes deployment of the application and CI/CD application configuration using Argo CD. 
The application defined in ArgoCD document deploys a chart from a GitHub repository to a Kubernetes cluster.

## Prerequisites
Before deploying the CI/CD application, ensure that you have the following prerequisites:<br/>
- ArgoCD is installed and running in your Kubernetes cluster.
- You have a **secret manifest file** with your database user and password in the same namespace specified in destination block.
- You have **docker-registry secret** with your AWS ECR creds in the same namespace specified in destination block.
- Update the **values.yaml** as shown in the **values-override.yaml**<br/><br/>

To apply the configuration provided in the ArgoCD YAML:
```
kubectl apply -f application.yaml
```
You can use the kubectl apply command or apply the configuration through the ArgoCD UI.
ArgoCD will automatically synchronize the application based on the sync policy. It will pull the chart from the provided repository, deploy it to the specified cluster and namespace, 
and ensure that it remains in the desired state.<br/><br/>

- [x] Link to source code of the application and IaC for AWS EKS Kuberneetes cluster: [source-project](https://github.com/Alexarikel/super-barnacle)
- [x] Link to Jenkinsfile for building the application and publishing the image to a registry: [jenkinsfile](https://github.com/Alexarikel/jenkins-for-super-barnacle/tree/main)


