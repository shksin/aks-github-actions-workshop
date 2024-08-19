name: Run Azure Login with OpenID Connect
on: [workflow_dispatch]

permissions:
  id-token: write # Require write permission to Fetch an OIDC token.
      
jobs: 
  test:
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