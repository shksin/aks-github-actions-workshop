Here's the updated step-by-step document using the new OpenID Connect (OIDC) approach and the `@azure/k8s-deploy` GitHub Action for performing a blue-green deployment in Azure Kubernetes Service (AKS) using the App Routing add-on:

---

# Lab: Performing Blue-Green Deployment in AKS using the App Routing Add-on and GitHub Actions with OIDC and `@azure/k8s-deploy`

## Objective:
This lab will guide you through the process of setting up a blue-green deployment strategy in Azure Kubernetes Service (AKS) using the App Routing add-on and GitHub Actions. By the end of this lab, you will have a CI/CD pipeline that automates the deployment of two environments (blue and green) and switches traffic between them using the App Routing add-on.

## Prerequisites:
1. **Azure AKS Cluster**: Deployed and configured with the App Routing add-on.
2. **GitHub Repository**: Set up with the existing deployment workflow.
3. **Azure CLI**: Installed and configured on your local machine.
4. **Kubectl**: Installed on your local machine.
5. **Basic Knowledge**: Familiarity with Kubernetes, GitHub Actions, and YAML syntax.

---

## Step 1: Enable the App Routing Add-on

1. **Enable the App Routing Add-on**:
   - Use the `--enable-app-routing` switch to enable the App Routing add-on in your AKS cluster:

     ```bash
     az aks create --resource-group myResourceGroup --name myAKSCluster --enable-app-routing --node-count 1 --generate-ssh-keys
     ```

   - If you already have an AKS cluster, you can enable the App Routing add-on using the following command:

     ```bash
     az aks update --resource-group myResourceGroup --name myAKSCluster --enable-app-routing
     ```

2. **Retrieve the Ingress Endpoint**:
   - After enabling the add-on, retrieve the external IP or hostname associated with the App Routing Ingress Controller:

     ```bash
     kubectl get svc -n kube-system
     ```

   - Look for the `app-routing-nginx-ingress` service to find the external IP or hostname, which you will use in the Ingress configuration.

---

## Step 2: Set Up the Initial Blue and Green Environments

1. **Create Separate Helm Values Files**:
   - In your repository, create separate Helm values files for the blue and green environments under the `k8s` directory:

     **`values-blue.yaml`**:
     ```yaml
     app:
       name: myapp-blue
       replicaCount: 2
       image:
         repository: nginx
         tag: latest
       service:
         name: myapp-blue-service
         port: 80
     ```

     **`values-green.yaml`**:
     ```yaml
     app:
       name: myapp-green
       replicaCount: 2
       image:
         repository: nginx
         tag: latest
       service:
         name: myapp-green-service
         port: 80
     ```

2. **Create Kubernetes Services**:
   - Ensure that both blue and green environments have their own services defined in the Helm chart templates:

     **`service.yaml`**:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: {{ .Values.app.service.name }}
     spec:
       selector:
         app: {{ .Values.app.name }}
       ports:
       - protocol: TCP
         port: {{ .Values.app.service.port }}
         targetPort: 80
       type: ClusterIP
     ```

---

## Step 3: Create an Ingress Resource for Traffic Switching

1. **Create an Ingress Resource**:
   - Create an `ingress.yaml` file in the `k8s` directory that will manage traffic between the blue and green services:

     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: myapp-ingress
       annotations:
         kubernetes.io/ingress.class: nginx
     spec:
       rules:
       - host: <subdomain>.<ingress-domain>
         http:
           paths:
           - path: /
             pathType: Prefix
             backend:
               service:
                 name: myapp-blue-service  # Initially route traffic to the blue service
                 port:
                   number: 80
     ```

   - Replace `<subdomain>` with a unique subdomain name for your app, and `<ingress-domain>` with the domain name associated with the `app-routing-nginx-ingress` service.

2. **Commit the Ingress Resource**:
   - Stage, commit, and push the `ingress.yaml` file to your GitHub repository:

     ```bash
     git add k8s/ingress.yaml
     git commit -m "Added ingress resource for blue-green deployment using AKS App Routing"
     git push origin main
     ```

---

## Step 4: Update GitHub Actions Workflow for Blue-Green Deployment

1. **Edit the `deploy-aks.yml` file** in the `.github/workflows` directory:

   ```yaml
   name: Blue-Green Deployment to AKS

   on:
     push:
       branches:
         - main

   permissions:
     id-token: write # Required for OIDC authentication
     contents: read  # Required to read repository contents

   jobs:
     deploy:
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

       - name: Deploy to Blue Environment
         uses: azure/k8s-deploy@v3
         with:
           manifests: |
             k8s/values-blue.yaml
           namespace: default
           images: nginx:latest

       - name: Deploy to Green Environment
         uses: azure/k8s-deploy@v3
         with:
           manifests: |
             k8s/values-green.yaml
           namespace: default
           images: nginx:latest

       - name: Switch Traffic to Green
         if: success()
         run: |
           kubectl patch ingress myapp-ingress -p '{"spec":{"rules":[{"host":"<subdomain>.<ingress-domain>","http":{"paths":[{"path":"/","pathType":"Prefix","backend":{"service":{"name":"myapp-green-service","port":{"number":80}}}}]}}]}}'
   ```

   - Replace `<subdomain>` and `<ingress-domain>` with the appropriate values.

   - This workflow deploys to both blue and green environments and then switches traffic to the green environment if the deployment is successful.

2. **Commit and Push the Workflow Changes**:
   - Stage, commit, and push the changes to your GitHub repository:

     ```bash
     git add .github/workflows/deploy-aks.yml
     git commit -m "Set up blue-green deployment pipeline with AKS App Routing"
     git push origin main
     ```

---

## Step 5: Trigger and Verify the Deployment

1. **Push Changes to Trigger the Workflow**:
   - Push any changes to the `main` branch to trigger the GitHub Actions workflow. This will deploy the application to both environments and switch the traffic to the green environment.

2. **Access Your Application**:
   - Access your application via the configured domain (`<subdomain>.<ingress-domain>`) to verify that the green environment is now serving traffic.

3. **Verify the Ingress Configuration**:
   - Run the following command to ensure that the ingress resource has been updated to route traffic to the green service:

     ```bash
     kubectl get ingress myapp-ingress -o yaml
     ```

---

## Step 6: Roll Back to the Blue Environment (If Necessary)

1. **Switch Traffic Back to Blue**:
   - If you need to roll back to the blue environment, you can manually patch the ingress resource:

     ```bash
     kubectl patch ingress myapp-ingress -p '{"spec":{"rules":[{"host":"<subdomain>.<ingress-domain>","http":{"paths":[{"path":"/","pathType":"Prefix","backend":{"service":{"name":"myapp-blue-service","port":{"number":80}}}}]}}]}}'
     ```

2. **Verify the Rollback**:
   - Access your application again to ensure that traffic is now routed back to the blue environment.

---

## Step 7: Clean Up Resources

1. **Delete the Blue and Green Services** (if no longer needed):
   - Clean up the blue and green services if you no longer need them:

     ```bash
     helm uninstall myapp-blue
     helm uninstall myapp-green
     ```

2. **Delete the Ingress Resource**:
   - Delete the ingress resource:

     ```bash
     kubectl delete ingress myapp-ingress
     ```

3. **Delete the Resource Group** (if no longer needed):
   - Clean up your Azure resources:

     ```bash
     az group delete --name myResourceGroup --yes --no-wait
     ```

---

## Conclusion

Congratulations! You have successfully set up a blue-green deployment strategy in AK