
# Lab: Integrating Azure Key Vault with AKS Using GitHub Actions

## Objective:
This lab will guide you through the process of creating an Azure Key Vault, storing a secret in it, and referencing that secret in an Azure Kubernetes Service (AKS) deployment using GitHub Actions. By the end of this lab, you'll be able to securely manage sensitive information (such as passwords, tokens, or keys) in your AKS deployments.

## Prerequisites:
1. **Azure Subscription**: Ensure you have an active Azure subscription.
2. **Azure AKS Cluster**: Deployed and configured.
3. **GitHub Repository**: Set up with the existing deployment workflow.
4. **Azure CLI**: Installed and configured on your local machine.
5. **Kubectl**: Installed on your local machine.
6. **Basic Knowledge**: Familiarity with Kubernetes, Azure, GitHub Actions, and YAML syntax.

---

## Step 1: Create an Azure Key Vault and Store a Secret

1. **Create an Azure Key Vault**:
   - Use the Azure CLI to create a Key Vault:

     ```bash
     az keyvault create --name myKeyVault --resource-group myResourceGroup --location eastus
     ```

   - Replace `myKeyVault` with a unique name for your Key Vault.

2. **Store a Secret in the Key Vault**:
   - Store a secret in your Key Vault:

     ```bash
     az keyvault secret set --vault-name myKeyVault --name mySecret --value "mySecretValue"
     ```

   - Replace `mySecret` with the name of your secret and `mySecretValue` with the secret value you want to store.

## Step 2: Assign Access Policy for AKS to Access Key Vault

1. **Retrieve the AKS Managed Identity**:
   - Get the managed identity of your AKS cluster:

     ```bash
     az aks show --resource-group myResourceGroup --name myAKSCluster --query identityProfile.kubeletidentity.clientId -o tsv
     ```

   - This will return the `clientId` of the managed identity used by the AKS cluster.

2. **Assign Key Vault Access Policy**:
   - Grant the AKS managed identity access to the Key Vault:

     ```bash
     az keyvault set-policy --name myKeyVault --spn <clientId> --secret-permissions get
     ```

   - Replace `<clientId>` with the `clientId` retrieved in the previous step.

## Step 3: Reference the Key Vault Secret in an AKS Deployment

1. **Create a Kubernetes Secret YAML File**:
   - In your repository, create a `k8s/secret.yaml` file to define a Kubernetes Secret that references the Azure Key Vault secret:

     ```yaml
     apiVersion: v1
     kind: Secret
     metadata:
       name: myapp-secret
     type: Opaque
     stringData:
       SECRET_VALUE: <secret-placeholder>
     ```

   - Replace `<secret-placeholder>` with a placeholder that will be replaced with the actual secret value during deployment.

2. **Update the Deployment YAML to Use the Secret**:
   - Modify your deployment YAML file (e.g., `k8s/deployment.yaml`) to use the secret as an environment variable:

     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: myapp
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
             env:
             - name: SECRET_VALUE
               valueFrom:
                 secretKeyRef:
                   name: myapp-secret
                   key: SECRET_VALUE
     ```

## Step 4: Update GitHub Actions Workflow to Inject the Secret

1. **Create a GitHub Actions Workflow File**:
   - Create a new workflow file named `aks-deployment-with-secret.yml` in the `.github/workflows` directory:

     ```yaml
     name: AKS Deployment with Azure Key Vault Secret

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

         - name: Log in to Azure
           uses: azure/login@v1
           with:
             creds: ${{ secrets.AZURE_CREDENTIALS }}

         - name: Set up Kubernetes tools
           uses: azure/setup-kubectl@v3
           with:
             version: 'latest'

         - name: Get AKS credentials
           run: |
             az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

         - name: Retrieve Secret from Key Vault
           id: get_secret
           run: |
             secret=$(az keyvault secret show --name mySecret --vault-name myKeyVault --query value -o tsv)
             echo "::set-output name=SECRET_VALUE::$secret"

         - name: Replace Secret in Kubernetes YAML
           run: |
             sed -i 's|<secret-placeholder>|${{ steps.get_secret.outputs.SECRET_VALUE }}|g' k8s/secret.yaml

         - name: Apply Kubernetes manifests
           run: |
             kubectl apply -f k8s/secret.yaml
             kubectl apply -f k8s/deployment.yaml
     ```

   - This workflow retrieves the secret from Azure Key Vault, injects it into the Kubernetes Secret manifest, and deploys the application to AKS.

2. **Commit and Push the Workflow**:
   - Stage, commit, and push the workflow file to your GitHub repository:

     ```bash
     git add .github/workflows/aks-deployment-with-secret.yml
     git commit -m "Added workflow to deploy AKS with Key Vault secret"
     git push origin main
     ```

## Step 5: Trigger the Deployment and Verify

1. **Push Changes to Trigger the Workflow**:
   - Any push to the `main` branch will trigger the workflow, retrieve the secret from Azure Key Vault, and deploy the application to AKS.

2. **Monitor the Workflow**:
   - Navigate to the Actions tab in your GitHub repository to monitor the progress of the workflow. Ensure that the secret is correctly retrieved and injected into the Kubernetes deployment.

3. **Verify the Deployment**:
   - Once the deployment is complete, verify that the application is running in AKS and the secret has been correctly applied.

## Step 6: Clean Up Resources

1. **Delete the Kubernetes Secret** (if no longer needed):
   - If you want to clean up the secret in Kubernetes:

     ```bash
     kubectl delete secret myapp-secret
     ```

2. **Delete the Key Vault** (if no longer needed):
   - Clean up your Azure resources by deleting the Key Vault:

     ```bash
     az keyvault delete --name myKeyVault
     ```

3. **Delete the Resource Group** (if no longer needed):
   - Clean up your Azure resources by deleting the resource group:

     ```bash
     az group delete --name myResourceGroup --yes --no-wait
     ```

---

## Conclusion

Congratulations! You have successfully integrated Azure Key Vault with your AKS deployment using GitHub Actions. This setup ensures that sensitive information such as passwords, tokens, or keys are securely managed and injected into your Kubernetes deployments.
