
# Lab: Creating an Azure Container Registry, Pushing an Image to It, and Deploying an App to AKS Using GitHub Actions

## Objective:
This lab will guide you through the process of creating an Azure Container Registry (ACR), pushing a Docker image to it, and deploying an application to Azure Kubernetes Service (AKS) using GitHub Actions. The goal is to use native GitHub Actions as much as possible, avoiding Azure CLI-based runs.

## Prerequisites:
1. **Azure Subscription**: An active Azure subscription.
2. **Azure AKS Cluster**: Deployed and configured.
3. **GitHub Repository**: Set up with the application code and existing deployment workflow.
4. **Docker**: Docker installed on your local machine for local testing (optional).
5. **Basic Knowledge**: Familiarity with Docker, Kubernetes, GitHub Actions, and YAML syntax.

---

## Step 1: Create an Azure Container Registry (ACR)

1. **Create an ACR via GitHub Actions**:
   - Add a new workflow file to your repository in the `.github/workflows` directory called `create-acr.yml`:

     ```yaml
     name: Create Azure Container Registry

     on:
  [workflow_dispatch]

    permissions:
      id-token: write # Require write permission to Fetch an OIDC token.
      contents: read # Required to read repository contents
      actions: read # Required for GitHub Actions
      checks: write # Required for status checks

    env:
          RESOURCE_GROUP: myResourceGroup
          AKS_CLUSTER_NAME: myAKSCluster
          ACR_NAME: myACRName

    jobs:
          create-acr:
            runs-on: ubuntu-latest

            steps:
            - name: Checkout code
              uses: actions/checkout@v2

            - name: Azure Login
              uses: azure/login@v2
              with:
                client-id: ${{ secrets.AZURE_CLIENT_ID }}
                tenant-id: ${{ secrets.AZURE_TENANT_ID }}
                subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }} 
        
            - name: Create Azure Container Registry
              uses: azure/cli@v2
              with:
                azcliversion: latest
                inlineScript: |
                  az acr create --resource-group ${{ env.RESOURCE_GROUP }} --name ${{ env.ACR_NAME }} --sku Basic

            - name: Attach ACR to AKS
              uses: azure/cli@v2
              with:
                azcliversion: latest
                inlineScript: |
                  az aks update --name ${{ env.AKS_CLUSTER_NAME }} --resource-group ${{ env.RESOURCE_GROUP }} --attach-acr ${{ env.ACR_NAME }}
     ```

   > **Note:** 
      - Replace `myResourceGroup` with the name of your Azure Resource Group created in Lab2.
      - Replace `myAKSCluster` with your AKS cluster created in Lab2.
      - Replace `myACRName` with the desired name for your Azure Container Registry.

2. **Run the Workflow**:
   - Commit and push the workflow file to the `main` branch. This will trigger the GitHub Actions workflow to create an ACR in your specified resource group.

---

## Step 2: Build and Push the Docker Image to ACR

1. **Create a Workflow to Log in, Build, and Push the Image**:
   - Add a new workflow file to your repository in the `.github/workflows` directory called `build-and-push.yml`:

     ```yaml
     name: Build and Push Docker Image to ACR

     on:
       push:
         branches:
           - main

     jobs:
       build-and-push:
         runs-on: ubuntu-latest

         steps:
         - name: Checkout code
           uses: actions/checkout@v2

         - name: Log in to Azure Container Registry
           uses: azure/azure-container-registry-login@v1
           with:
             login-server: ${{ secrets.ACR_NAME }}.azurecr.io
             client-id: ${{ secrets.AZURE_CLIENT_ID }}
             tenant-id: ${{ secrets.AZURE_TENANT_ID }}
             client-secret: ${{ secrets.AZURE_CLIENT_SECRET }}

         - name: Build and Push Docker Image
           uses: azure/azure-container-registry-build@v1
           with:
             azure-subscription: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
             registry: ${{ secrets.ACR_NAME }}
             images: |
               myapp:${{ github.sha }}
     ```

   - Replace `ACR_NAME`, `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_CLIENT_SECRET`, and `AZURE_SUBSCRIPTION_ID` with the appropriate secrets stored in your GitHub repository.

2. **Run the Workflow**:
   - Commit and push the workflow file to the `main` branch. This will trigger the GitHub Actions workflow to log in, build the Docker image, and push it to the Azure Container Registry.

---

## Step 3: Deploy the Application to AKS Using GitHub Actions

1. **Create a Workflow to Deploy the Application**:
   - Add a new workflow file to your repository in the `.github/workflows` directory called `deploy-to-aks.yml`:

     ```yaml
     name: Deploy to AKS

     on:
       push:
         branches:
           - main

     jobs:
       deploy:
         runs-on: ubuntu-latest

         steps:
         - name: Checkout code
           uses: actions/checkout@v2

         - name: Set up Kubernetes tools
           uses: azure/setup-kubectl@v3
           with:
             version: 'latest'

         - name: Log in to Azure
           uses: azure/login@v1
           with:
             creds: ${{ secrets.AZURE_CREDENTIALS }}

         - name: Get AKS Credentials
           uses: azure/aks-set-context@v1
           with:
             resource-group: ${{ secrets.AZURE_RESOURCE_GROUP }}
             cluster-name: ${{ secrets.AKS_CLUSTER_NAME }}

         - name: Deploy to AKS
           run: |
             kubectl set image deployment/myapp myapp=${{ secrets.ACR_NAME }}.azurecr.io/myapp:${{ github.sha }}
             kubectl rollout status deployment/myapp
     ```

   - Replace `AZURE_RESOURCE_GROUP`, `AKS_CLUSTER_NAME`, and `ACR_NAME` with the appropriate secrets stored in your GitHub repository.

2. **Run the Workflow**:
   - Commit and push the workflow file to the `main` branch. This will trigger the GitHub Actions workflow to deploy the application to AKS.

---

## Step 4: Verify the Deployment

1. **Check the Deployment**:
   - After the workflow completes, verify the deployment by accessing your application through the AKS service's external IP address or configured domain.

2. **Monitor the Application**:
   - Use `kubectl get pods` and `kubectl get services` to monitor the status of your application and ensure everything is running as expected.

---

## Step 5: Clean Up Resources (Optional)

1. **Delete the Azure Container Registry**:
   - If you no longer need the Azure Container Registry, you can delete it using the Azure portal or CLI.

2. **Delete the AKS Cluster**:
   - If you no longer need the AKS cluster, you can delete it using the Azure portal or CLI to avoid incurring costs.

---

## Conclusion

Congratulations! You have successfully created an Azure Container Registry, pushed a Docker image to it, and deployed an application to Azure Kubernetes Service using GitHub Actions. This setup automates the entire process, from image creation to deployment, using native GitHub Actions.
