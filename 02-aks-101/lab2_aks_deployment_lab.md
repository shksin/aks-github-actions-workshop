# Lab2: Deploying an AKS Cluster using GitHub Actions with OpenID Connect (OIDC)

## Objective:
This lab will guide you through the process of deploying an Azure Kubernetes Service (AKS) cluster using GitHub Actions with OpenID Connect (OIDC) for authentication. By the end of this lab, you'll have an AKS cluster deployed and managed via a CI/CD pipeline on GitHub Actions.

## Prerequisites:
1. **Basic Knowledge**: Familiarity with Kubernetes, GitHub Actions, and YAML syntax. Ensure you have completed [Lab0-GitHub Actions 101](../01-gh-actions-101/readme.md) and have established successful connectivity between GitHub Actions and Azure.
**Azure CLI**: Installed and configured on your local machine. [How to install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
**Kubectl**: Installed on your local machine. [How to install the Kubectl](https://kubernetes.io/docs/tasks/tools/)

---

## Step 1: Clone the GitHub Repository to Your Local Machine

1. **Clone the repository** to your local machine:
   ```bash
   git clone <url of your repository>

## Step 2: Create Kubernetes Deployment Manifest

1. **In the root folder of your repository**, create a directory for Kubernetes manifests. Let's call it `k8s`:
   ```bash
   mkdir -p k8s
   ```

2. **Create a deployment YAML file**:
   ```bash
   touch k8s/deployment.yml
   ```
   
3. **Edit the deployment YAML file** with the following content:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-deployment
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: myapp
     template:
       metadata:
         labels:
           app: myapp
       spec:
         containers:
         - name: myapp
           image: nginx:latest
           ports:
           - containerPort: 80
   ```

## Step 3: Create a GitHub Actions Workflow File

1. **In your repository**, navigate to the `.github/workflows` directory: 

2. **Create a new YAML file**:
    ```bash
    touch .github/workflows/deploy-aks.yml
    ```

  *Alternatively, you can create the file manually in the `.github/workflows` directory*

3. **Edit the YAML file** with the following content:

   ```yaml
   name: Deploy AKS

   on:
     push:
       branches:
         - main

   permissions:
     id-token: write # Required for OIDC authentication
     contents: read # Required to read repository contents

   env:
      RESOURCE_GROUP: myResourceGroup
      AKS_CLUSTER_NAME: myAKSCluster
      LOCATION: australiaeast

   jobs:
     build:
       runs-on: ubuntu-latest

       steps:
       - name: Checkout the code
         uses: actions/checkout@v2

       - name: Azure Login
         uses: azure/login@v2
         with:
           client-id: ${{ secrets.AZURE_CLIENT_ID }}
           tenant-id: ${{ secrets.AZURE_TENANT_ID }}
           subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
           enable-AzPSSession: true

       - name: Set up Kubernetes tools
         uses: azure/setup-kubectl@v3
         with:
           version: 'latest'

       - name: Set up Helm
         uses: azure/setup-helm@v1
         with:
           version: 'latest'

       - name: Create Resource Group
         run: |
           az group create --name $RESOURCE_GROUP --location $LOCATION

       - name: Create AKS Cluster
         run: |
           az aks create --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER_NAME --node-count 1 --enable-addons monitoring --generate-ssh-keys

       - name: Get AKS Credentials
         run: |
           az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER_NAME

       - name: Deploy to AKS
         run: |
           kubectl apply -f k8s/deployment.yml
   ```

      > **Note:** 
      - Replace `myResourceGroup` with your desired resource group name.
      - Replace `myAKSCluster` with your desired AKS cluster name.
      - Provide the correct `LOCATION` for your Azure region.
      - Ensure that the `k8s/deployment.yml`file paths exist in your repository.


## Step 4: Push the Changes to GitHub

1. **Stage, commit, and push** your changes:
   ```bash
   git add .
   git commit -m "Set up AKS deployment workflow"
   git push origin main
   ```

2. **Monitor the GitHub Actions Workflow**:
   - Navigate to the Actions tab of your GitHub repository.
   - Watch the workflow as it runs, which will create the AKS cluster and deploy the application.

## Step 5: Verify the Deployment
1. **Login to Azure**:
   - Run the following command to login to Azure:
     ```bash
     az login
     ```

2. **Get the AKS Cluster Credentials**:
   - Run the following command to get the AKS cluster credentials:
     ```bash
     az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
     ```
     > **Note:** Replace `myResourceGroup` with the    correct resource group name .
      Replace `myAKSCluster` with the correct AKS cluster name.
3. **Access the AKS Cluster**:
   - Run the following command to ensure you have the correct context:
     ```bash
     kubectl get nodes
     ```

4. **Verify the Deployment**:
   - Check the status of your deployment:
     ```bash
     kubectl get deployments
     ```

   - Check the running pods:
     ```bash
     kubectl get pods
     ```

3. **Expose the Deployment (Optional)**:
   - To access the application, you may want to expose it as a service:
     ```bash
     kubectl expose deployment myapp-deployment --type=LoadBalancer --name=myapp-service
     ```

## Step 6: Clean Up Resources

1. **Delete the Resource Group** (if no longer needed):
   ```bash
   az group delete --name $RESOURCE_GROUP --yes --no-wait
   ```

---

## Conclusion

Congratulations! You have successfully deployed an AKS cluster using GitHub Actions with OpenID Connect (OIDC). This setup can be further expanded and customized to suit more complex CI/CD pipelines and Kubernetes deployments.

---

This updated document now uses OIDC for secure authentication, eliminating the need to store secrets in GitHub for Azure authentication.
