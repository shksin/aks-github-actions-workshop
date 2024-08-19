
# Lab: Performing Blue-Green Deployment in AKS using GitHub Actions

## Objective:
This lab will guide you through the process of setting up a blue-green deployment strategy in Azure Kubernetes Service (AKS) using GitHub Actions. By the end of this lab, you will have a CI/CD pipeline that automates the deployment of two environments (blue and green) and switches traffic between them.

## Prerequisites:
1. **Azure AKS Cluster**: Deployed and configured.
2. **GitHub Repository**: Set up with the existing deployment workflow.
3. **Azure CLI**: Installed and configured on your local machine.
4. **Kubectl**: Installed on your local machine.
5. **Basic Knowledge**: Familiarity with Kubernetes, Helm, GitHub Actions, and YAML syntax.

---

## Step 1: Set Up the Initial Blue and Green Environments

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

## Step 2: Create an Ingress Resource for Traffic Switching

1. **Create an Ingress Resource**:
   - Create an `ingress.yaml` file in the `k8s` directory that will manage traffic between the blue and green services:

     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: myapp-ingress
       annotations:
         nginx.ingress.kubernetes.io/rewrite-target: /
     spec:
       rules:
       - host: myapp.example.com
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

2. **Commit the Ingress Resource**:
   - Stage, commit, and push the `ingress.yaml` file to your GitHub repository:

     ```bash
     git add k8s/ingress.yaml
     git commit -m "Added ingress resource for blue-green deployment"
     git push origin main
     ```

---

## Step 3: Update GitHub Actions Workflow for Blue-Green Deployment

1. **Edit the `deploy-aks.yml` file** in the `.github/workflows` directory:

   ```yaml
   name: Blue-Green Deployment to AKS

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

       - name: Get AKS Credentials
         run: |
           az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

       - name: Deploy to Blue Environment
         run: |
           helm upgrade --install myapp-blue ./myapp -f k8s/values-blue.yaml --set app.image.tag=${{ github.sha }}

       - name: Deploy to Green Environment
         run: |
           helm upgrade --install myapp-green ./myapp -f k8s/values-green.yaml --set app.image.tag=${{ github.sha }}

       - name: Switch Traffic to Green
         if: success()
         run: |
           kubectl patch ingress myapp-ingress -p '{"spec":{"rules":[{"host":"myapp.example.com","http":{"paths":[{"path":"/","pathType":"Prefix","backend":{"service":{"name":"myapp-green-service","port":{"number":80}}}}]}}]}}'
   ```

   - This workflow deploys to both blue and green environments and then switches traffic to the green environment if the deployment is successful.

2. **Commit and Push the Workflow Changes**:
   - Stage, commit, and push the changes to your GitHub repository:

     ```bash
     git add .github/workflows/deploy-aks.yml
     git commit -m "Set up blue-green deployment pipeline"
     git push origin main
     ```

---

## Step 4: Trigger and Verify the Deployment

1. **Push Changes to Trigger the Workflow**:
   - Push any changes to the `main` branch to trigger the GitHub Actions workflow. This will deploy the application to both environments and switch the traffic to the green environment.

2. **Access Your Application**:
   - Access your application via the configured domain (`myapp.example.com`) to verify that the green environment is now serving traffic.

3. **Verify the Ingress Configuration**:
   - Run the following command to ensure that the ingress resource has been updated to route traffic to the green service:

     ```bash
     kubectl get ingress myapp-ingress -o yaml
     ```

---

## Step 5: Roll Back to the Blue Environment (If Necessary)

1. **Switch Traffic Back to Blue**:
   - If you need to roll back to the blue environment, you can manually patch the ingress resource:

     ```bash
     kubectl patch ingress myapp-ingress -p '{"spec":{"rules":[{"host":"myapp.example.com","http":{"paths":[{"path":"/","pathType":"Prefix","backend":{"service":{"name":"myapp-blue-service","port":{"number":80}}}}]}}]}}'
     ```

2. **Verify the Rollback**:
   - Access your application again to ensure that traffic is now routed back to the blue environment.

---

## Step 6: Clean Up Resources

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

Congratulations! You have successfully set up a blue-green deployment strategy in AKS using GitHub Actions. This deployment strategy allows you to minimize downtime and reduce the risk of introducing issues during deployments by having two environments (blue and green) that you can easily switch between.
