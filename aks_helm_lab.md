
# Lab: Packaging and Deploying an App using Helm with GitHub Actions

## Objective:
This lab will guide you through the process of packaging your application as a Helm chart and deploying it to an Azure Kubernetes Service (AKS) cluster using GitHub Actions. By the end of this lab, you will have a Helm chart for your application and a CI/CD pipeline that automatically deploys it using Helm.

## Prerequisites:
1. **Azure AKS Cluster**: Deployed and configured.
2. **GitHub Repository**: Set up with the existing deployment workflow.
3. **Azure CLI**: Installed and configured on your local machine.
4. **Kubectl and Helm**: Installed on your local machine.
5. **Basic Knowledge**: Familiarity with Kubernetes, Helm, GitHub Actions, and YAML syntax.

---

## Step 1: Install Helm and Create a Helm Chart

1. **Install Helm**:
   - Ensure Helm is installed on your local machine. You can install Helm using the following command:

     ```bash
     curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
     ```

2. **Create a Helm Chart**:
   - Navigate to your project directory and create a Helm chart:

     ```bash
     helm create myapp
     ```

   - This command generates a directory structure for your Helm chart, including a default `values.yaml` file, templates for Kubernetes resources, and more.

3. **Customize the Helm Chart**:
   - Navigate to the `myapp` directory and customize the templates according to your application's requirements.
   - For example, update `deployment.yaml` in the `templates` directory to reflect your application's deployment specifics:

     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: {{ .Values.app.name }}
     spec:
       replicas: {{ .Values.app.replicaCount }}
       selector:
         matchLabels:
           app: {{ .Values.app.name }}
       template:
         metadata:
           labels:
             app: {{ .Values.app.name }}
           spec:
             containers:
             - name: {{ .Values.app.name }}
               image: {{ .Values.app.image.repository }}:{{ .Values.app.image.tag }}
               ports:
               - containerPort: {{ .Values.app.service.port }}
     ```

4. **Update `values.yaml`**:
   - Define default values in the `values.yaml` file:

     ```yaml
     app:
       name: myapp
       replicaCount: 2
       image:
         repository: nginx
         tag: latest
       service:
         port: 80
     ```

5. **Test the Helm Chart Locally**:
   - To ensure that your Helm chart is working as expected, you can test it locally by running:

     ```bash
     helm install --dry-run --debug myapp ./myapp
     ```

   - This will simulate the deployment and display the generated Kubernetes manifests without actually deploying them.

## Step 2: Push the Helm Chart to GitHub Repository

1. **Commit the Helm Chart**:
   - Stage, commit, and push the Helm chart to your GitHub repository:

     ```bash
     git add myapp
     git commit -m "Added Helm chart for myapp"
     git push origin main
     ```

## Step 3: Update GitHub Actions Workflow for Helm Deployment

Now, you'll update your GitHub Actions workflow to use Helm for deploying the application.

1. **Edit the `deploy-aks.yml` file** in the `.github/workflows` directory:

   ```yaml
   name: Deploy AKS with Helm

   on:
     push:
       branches:
         - main

   jobs:
     build:
       runs-on: ubuntu-latest

       steps:
       - name: Checkout the code
         uses: actions/checkout@v2

       - name: Log in to Azure
         uses: azure/login@v1
         with:
           creds: ${{ secrets.AZURE_CREDENTIALS }}

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

       - name: Deploy Helm Chart
         run: |
           helm upgrade --install myapp ./myapp --set app.image.tag=${{ github.sha }}
   ```

   - **Helm Upgrade/Install**: The `helm upgrade --install` command will install the Helm chart if it's not already deployed or upgrade it if it is.

2. **Commit and Push the Workflow Changes**:
   - Stage, commit, and push the changes to your GitHub repository:

     ```bash
     git add .github/workflows/deploy-aks.yml
     git commit -m "Updated GitHub Actions workflow to deploy with Helm"
     git push origin main
     ```

## Step 4: Trigger the GitHub Actions Workflow

1. **Push Changes to Trigger the Workflow**:
   - Push any changes to the `main` branch to trigger the GitHub Actions workflow. This will run the workflow defined in your `deploy-aks.yml` file, which will deploy the application using Helm.

2. **Monitor the GitHub Actions Workflow**:
   - Navigate to the Actions tab of your GitHub repository.
   - Watch the workflow as it runs, deploying your application using Helm.

## Step 5: Verify the Deployment

1. **Verify the Helm Release**:
   - Check the Helm release to ensure it's successfully deployed:

     ```bash
     helm list
     ```

2. **Verify the Application**:
   - Check the status of your deployment:

     ```bash
     kubectl get deployments
     ```

   - Check the running pods:

     ```bash
     kubectl get pods
     ```

   - Access the service to verify that your application is running correctly.

## Step 6: Clean Up Resources

1. **Uninstall the Helm Release**:
   - If you no longer need the application, you can uninstall the Helm release:

     ```bash
     helm uninstall myapp
     ```

2. **Delete the Resource Group** (if no longer needed):
   - Clean up your Azure resources:

     ```bash
     az group delete --name myResourceGroup --yes --no-wait
     ```

---

## Conclusion

Congratulations! You have successfully packaged your application using Helm and deployed it to an AKS cluster via GitHub Actions. This setup provides a powerful and flexible way to manage Kubernetes deployments, with the added benefits of Helm's templating and versioning capabilities.
