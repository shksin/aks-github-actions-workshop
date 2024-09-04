# AKS with GitHub Actions Workshop

## [Lab #1 - Getting Started with GitHub Actions](01-gh-actions-101/lab1-azure-oidc-login.md)

This is introduction level (101) workshop for those of you who are not familiar with GitHub Actions. This workshop covers the fundamentals of using GitHub Actions and you will learn how to:
 * Setup GitHub Actions workflow
 * Authenticate to Azure using OpenID Connect (OIDC) authentication.
 * Run some basic Azure CLI/Powershell commands to validate successful login.
> **Caution:**: Once you use GitHub Actions, you get addicted to it :)

 ## [Lab #2 - AKS Cluster Creation and kubectl command in GitHub Actions Workflow](02-aks-101/lab2_aks_deployment_lab.md)

This is introduction level (101) workshop for those of you who are new to AKS. We expect that you have some basic understanding of containerization concepts and know what container and container images are. This workshop covers the basics of Azure Kubernetes Service and you will learn how to
 * Provision a basic AKS cluster using GitHub Actions
 * Create an AKS deployment manifest that creates Nginx deployment and deploy it to the AKS cluster using ```kubectl``` commands in GitHub Actions workflow.
 * Verify the application deployment using ```kubectl``` commands


 ## [Lab #3 - Deploy an application to AKS using ```@azure/k8s-deploy``` module in GitHub Actions Workflow](03-app-deploy-aks/lab3_aks_nginx_deployment.md)

This workshop covers the basics of Azure Kubernetes Service and make you familiar with using GitHub Actions for AKS deployments. You will learn how to:  
 * Deploy an Nginx application the AKS Cluster utilizing the `@azure/k8s-deploy` module in GitHub Actions workflow.
 * Expose Ngixn deployment as a service to access the application

 ## [Lab #5 - Publish and deploy an application from Azure Container Registry (ACR) to AKS (Azure Kubernetes Service) using ```@azure/k8s-deploy``` module in GitHub Actions Workflow](04-app-deploy-acr-aks/lab4_acr_aks_deployment.md)

This workshop covers the basics of Azure Kubernetes Service and make you familiar with using GitHub Actions for AKS deployments. You will learn how to:
 * Build a Docker image and push it to Azure Container Registry (ACR) using GitHub Actions  
 * Deploy an App to AKS from ACR using GitHub Actions utilizing the `@azure/k8s-deploy` module in GitHub Actions workflow.

## [Lab #5 (Optional) - Advanced AKS Scenarios with GitHub Actions](05-advanced-aks-with-gh-actions/readme.md) 
This workshop covers the advanced scenarios of AKS with GitHub Actions. Each of the scenarios are independent and can be completed without any dependancy on each other. In this advanced lab you will learn how to: 
    * 
    * Integrate Azure Key Vault with AKS Using GitHub Actions
    * Package and Deploy an App using Helm with GitHub Actions
    * Scale a Deployed App Using Horizontal Pod Autoscaler (HPA) with GitHub Actions using OIDC

 

 The workshops consists of 8 labs (4 Basic and 4 advanced) and estimated time is 2 hours.
