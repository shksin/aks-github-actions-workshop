Here's the updated document that uses the OpenID Connect (OIDC) approach for deploying an AKS cluster using GitHub Actions:

---

# Lab: Deploying an AKS Cluster using GitHub Actions with OpenID Connect (OIDC)

## Objective:
This lab will guide you through the process of deploying an Azure Kubernetes Service (AKS) cluster using GitHub Actions with OpenID Connect (OIDC) for authentication. By the end of this lab, you'll have an AKS cluster deployed and managed via a CI/CD pipeline on GitHub Actions.

## Prerequisites:
1. **Azure Account**: Ensure you have an active Azure subscription.
2. **GitHub Account**: You should have a GitHub account with a repository ready to use.
3. **Azure CLI**: Installed and configured on your local machine.
4. **Kubectl**: Installed on your local machine.
5. **Basic Knowledge**: Familiarity with Kubernetes, GitHub Actions, and YAML syntax.

---

## Step 1: Prepare for Azure Authentication using OIDC

Using OpenID Connect (OIDC), GitHub Actions can securely authenticate with Azure without needing to store long-lived cloud secrets.

### Azure Setup:
1. **Create an Azure AD App Registration** (If you don't already have one):
   - In the Azure portal, navigate to "Azure Active Directory" > "App registrations" > "New registration".
   - Name your application (e.g., `GitHubActions-Deployment`), and register it.
   - Note the **Client ID** and **Tenant ID**; you'll need these later.

2. **Configure API Permissions**:
   - In the App Registration, go to "API permissions" > "Add a permission".
   - Select "Azure Service Management" > "Delegated permissions" > Select "user_impersonation" and add it.
   - Grant admin consent for your organization.

3. **Assign RBAC Roles**:
   - Assign the necessary Azure roles (e.g., Contributor) to the service principal created by the app registration.
   - This can be done using the Azure portal or via Azure CLI:
     ```bash
     az role assignment create --assignee <CLIENT_ID> --role Contributor --scope /subscriptions/YOUR_SUBSCRIPTION_ID
     ```
   - Replace `<CLIENT_ID>` with your App Registration's Client ID and `YOUR_SUBSCRIPTION_ID` with your Azure subscription ID.

### GitHub Setup:
1. **Add GitHub Secrets**:
   - Navigate to your GitHub repository.
   - Go to "Settings" > "Secrets and variables" > "Actions".
   - Add the following secrets:
     - `AZURE_CLIENT_ID`: Your Azure App Registration's Client ID.
     - `AZURE_TENANT_ID`: Your Azure Tenant ID.
     - `AZURE_SUBSCRIPTION_ID`: Your Azure Subscription ID.

## Step 2: Create a GitHub Actions Workflow File

1. **In your repository**, create a directory for GitHub Actions workflows:
   ```bash
   mkdir -p .github/workflows
   ```

2. **Create a new YAML file**:
   ```bash
   touch .github/workflows/deploy-aks.yml
   ```

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
           az group create --name myResourceGroup --location eastus

       - name: Create AKS Cluster
         run: |
           az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 1 --enable-addons monitoring --generate-ssh-keys

       - name: Get AKS Credentials
         run: |
           az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

       - name: Deploy to AKS
         run: |
           kubectl apply -f k8s/deployment.yml
   ```

4. **Replace placeholders**:
   - Replace `myResourceGroup` with your desired resource group name.
   - Replace `myAKSCluster` with your desired AKS cluster name.
   - Ensure that the `k8s/deployment.yml` file path exists in your repository.

## Step 3: Create Kubernetes Deployment Manifest

1. **In your repository**, create a directory for Kubernetes manifests:
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

1. **Access the AKS Cluster**:
   - Run the following command to ensure you have the correct context:
     ```bash
     kubectl get nodes
     ```

2. **Verify the Deployment**:
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
   az group delete --name myResourceGroup --yes --no-wait
   ```

---

## Conclusion

Congratulations! You have successfully deployed an AKS cluster using GitHub Actions with OpenID Connect (OIDC). This setup can be further expanded and customized to suit more complex CI/CD pipelines and Kubernetes deployments.

---

This updated document now uses OIDC for secure authentication, eliminating the need to store secrets in GitHub for Azure authentication.