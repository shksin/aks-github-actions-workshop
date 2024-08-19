Here is the updated step-by-step document using the new OpenID Connect (OIDC) approach and the `@azure/k8s-deploy` GitHub Action for scaling a deployed app using Horizontal Pod Autoscaler (HPA) in Azure Kubernetes Service (AKS):

---

# Lab: Scaling a Deployed App Using Horizontal Pod Autoscaler (HPA) with GitHub Actions using OIDC and `@azure/k8s-deploy`

## Objective:
This lab will guide you through the process of setting up Horizontal Pod Autoscaler (HPA) for a deployed application in an Azure Kubernetes Service (AKS) cluster using GitHub Actions. By the end of this lab, you will have an automated CI/CD pipeline that not only deploys an application but also configures HPA to scale the application based on CPU usage.

## Prerequisites:
1. **Azure AKS Cluster**: Deployed and configured, ideally through the previous lab.
2. **GitHub Repository**: Set up with the existing deployment workflow.
3. **Azure CLI**: Installed and configured on your local machine.
4. **Kubectl**: Installed on your local machine.
5. **Basic Knowledge**: Familiarity with Kubernetes, GitHub Actions, and YAML syntax.

---

## Step 1: Modify the Kubernetes Deployment Manifest for HPA

To use HPA, your Kubernetes deployment needs to include resource requests and limits for CPU (and optionally memory). These resources will be used by the HPA to determine when to scale your application.

1. **Edit the `deployment.yml` file** in the `k8s` directory of your repository:

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
           resources:
             requests:
               cpu: "100m"
             limits:
               cpu: "200m"
   ```

   - **Resource Requests**: The minimum amount of CPU that the container needs.
   - **Resource Limits**: The maximum amount of CPU that the container can use.

2. **Commit and push the changes** to your GitHub repository:
   ```bash
   git add k8s/deployment.yml
   git commit -m "Updated deployment manifest with resource requests and limits"
   git push origin main
   ```

## Step 2: Create a Horizontal Pod Autoscaler YAML Manifest

Next, you'll create a YAML manifest for the HPA, which will define how Kubernetes should scale your application based on CPU usage.

1. **Create a new HPA YAML file** in the `k8s` directory:

   ```bash
   touch k8s/hpa.yml
   ```

2. **Edit the `hpa.yml` file** with the following content:

   ```yaml
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: myapp-hpa
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: myapp-deployment
     minReplicas: 2
     maxReplicas: 10
     metrics:
     - type: Resource
       resource:
         name: cpu
         target:
           type: Utilization
           averageUtilization: 50
   ```

   - **minReplicas**: The minimum number of pods to run.
   - **maxReplicas**: The maximum number of pods that can be scaled up to.
   - **averageUtilization**: The target CPU utilization percentage that will trigger scaling.

3. **Commit and push the changes** to your GitHub repository:
   ```bash
   git add k8s/hpa.yml
   git commit -m "Added HPA configuration"
   git push origin main
   ```

## Step 3: Update GitHub Actions Workflow to Apply HPA Configuration

You need to update your GitHub Actions workflow to apply the HPA configuration after deploying the application.

1. **Edit the `deploy-aks.yml` file** in the `.github/workflows` directory:

   ```yaml
   name: Deploy AKS with HPA

   on:
     push:
       branches:
         - main

   permissions:
     id-token: write # Required for OIDC authentication
     contents: read  # Required to read repository contents

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

       - name: Deploy to AKS
         uses: azure/k8s-deploy@v3
         with:
           manifests: |
             k8s/deployment.yml

       - name: Apply HPA Configuration
         uses: azure/k8s-deploy@v3
         with:
           manifests: |
             k8s/hpa.yml
   ```

2. **Commit and push the changes** to your GitHub repository:
   ```bash
   git add .github/workflows/deploy-aks.yml
   git commit -m "Updated GitHub Actions workflow to apply HPA"
   git push origin main
   ```

## Step 4: Monitor the HPA in Action

After the pipeline runs, your application should be deployed with the HPA configured. You can monitor the HPA and see it in action.

1. **Check the status of the HPA**:
   ```bash
   kubectl get hpa
   ```

2. **Check the status of the pods**:
   ```bash
   kubectl get pods
   ```

3. **Simulate a load** to see the HPA in action:
   - You can simulate load using tools like `kubectl run` or an external load testing tool to generate traffic.

   Example:
   ```bash
   kubectl run -i --tty load-generator --rm --image=busybox -- /bin/sh -c "while true; do wget -q -O- http://myapp-service; done"
   ```

4. **Watch the HPA scale the deployment**:
   - Monitor the number of replicas as the load increases, and observe how the HPA automatically adjusts the number of running pods based on the CPU utilization.

## Step 5: Clean Up Resources

1. **Delete the HPA and Deployment** (if no longer needed):
   ```bash
   kubectl delete hpa myapp-hpa
   kubectl delete deployment myapp-deployment
   ```

2. **Delete the Resource Group** (if no longer needed):
   ```bash
   az group delete --name myResourceGroup --yes --no-wait
   ```

---

## Conclusion

Congratulations! You have successfully configured and applied a Horizontal Pod Autoscaler (HPA) to scale your deployed application using GitHub Actions with OIDC and `@azure/k8s-deploy`. This automation ensures that your application can handle varying loads by scaling up or down as needed.

---

This updated guide now integrates the OIDC-based authentication and the `@azure/k8s-deploy` action to manage deployments and HPA configuration in an AKS environment using GitHub Actions.