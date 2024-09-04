# Lab1 - Azure Login with OpenID Connect in GitHub Actions

## Objective
In this lab, you will learn how toget started with GitHub Actions by creating an Actions workflow to invoke tasks in a job. The workflow will perfoorm following tasks :

1. Login to Azure using OpenID Connect (OIDC) authentication.
2. Run some basic Azure CLI/Powershell commands to validate successful login.


## Prerequisites

To use the Azure Login action with OIDC, you need to configure a federated identity credential on a Microsoft Entra application or a user-assigned managed identity. In this lab we will look at the Microsoft Entra application approach.

### Set up a GitHub Repository for the Lab
Follow the steps below to fork the given GitHub Repository [KCD-AKS-GitHubActions-Workshop](https://github.com/shksin/kcd-gha-aks-workshop)
## steps to fork the repository
1. Navigate to the [KCD-AKS-GitHubActions-Workshop](https://github.com/shksin/kcd-gha-aks-workshop)
2. Click on the Fork button on the top right corner of the page.
3. Select your GitHub account to fork the repository to your account.
 - [How to Fork a GitHub Repository](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo#forking-a-repository).


### Setup a Microsoft Entra Application

1. **Create a Microsoft Entra Application**  
   You can create a Microsoft Entra application with a service principal using Azure Portal : 
    - Navigate to the Azure Portal and search for 'Azure EntraId'
    - Click on 'App Registrations' and then 'New Registration'

   ![Create App Registration](assets/create-app-registration.png)

2. **Obtain the Following Values:**
   - *Client ID*
   - *Subscription ID* - This is the subscription you will use to deploy your resources later
   - *Directory (tenant) ID*

   ![SelectClientId](assets/select-clientid-tenantId.png)

   > **Note:** Make sure to store these values securely, as they will be used later in the GitHub Actions workflow to authenticate to Azure.

3. **Configure a Federated Identity Credential**
      Set up a federated identity credential on the Microsoft Entra application to trust tokens issued by GitHub Actions for your GitHub repository.

      Navigate to the 'Certificates & Secrets' section of the App Registration created above and add a new Federated Identity Credential.
    ![Fed Cred Menu](assets/federated%20credentials%20menu.png)
    ![Fed Cred](assets/federated%20credentials.png)

4. **Assign 'Contributor' Role to the Microsoft Entra Application for the Subscription**  
   Use the Azure Portal, assign 'Contributor' role to the Microsoft Entra Application created above for your Azure Subscription.
    - Select your Azure Subscription
    - Click on 'Access Control (IAM)' and then 'Add Role Assignment'
   ![Assign Role](assets/AddContributorRoleToSubscription.png)


5. **Fetch values stored in Step 2 and add them as Secrets to the GitHub Repo as following:**
   - *AZURE_CLIENT_ID*
   - *AZURE_SUBSCRIPTION_ID*
   - *AZURE_TENANT_ID*

   ![SelectClientId](assets/GHActionsSecret.png)

6. **Navigate to your GitHub Repository, select Action from the top menu and Add a workflow file:**

    ![Add Workflow](assets/setupworkflow.png)

7. **Create a new workflow file and add copy the content of the yaml file in following lab to authenticate GitHub Actions workflow with Azure**

```yaml
name: Run Azure Login with OpenID Connect
on: [workflow_dispatch]

permissions:
  id-token: write # Require write permission to Fetch an OIDC token.
      
jobs: 
  azure-oidc-login:
    runs-on: ubuntu-latest
    steps:
    - name: Azure Login
      uses: azure/login@v2
      with:
        client-id: ${{ secrets.AZURE_CLIENT_ID }}
        tenant-id: ${{ secrets.AZURE_TENANT_ID }}
        subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }} 
        enable-AzPSSession: true
    
    - name: Azure CLI script
      uses: azure/cli@v2
      with:
        azcliversion: latest
        inlineScript: |
          az account show
          # You can write your Azure CLI inline scripts here.
          az group list -o table

    - name: Azure PowerShell script
      uses: azure/powershell@v2
      with:
        azPSVersion: latest
        inlineScript: |
          Get-AzContext  
          # You can write your Azure PowerShell inline scripts here.
          Get-AzResourceGroup
```


7. **Commit the changes to the repository and navigate to the Actions tab to see the workflow in action**

  _Select the workflow and trigger it manually_:
    ![Manual Workflow Tab](assets/manualworkflow%20trigger.png)


8. **Navigate to the workflow run to see the logs and verify that workflow has authenticated with Azure and has run the CLI and PowerShell scripts successfully: **

    ![Workflow Logs](assets/SuccessfulWorkflow.png)


