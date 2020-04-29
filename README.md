# GitHub Action for Azure Resource Manager


With ARM GitHub Action, you can automate your workflow by executing [Azure CLI](https://github.com/Azure/azure-cli) commands to deploy ARM templates and manage Azure resources.

The action executes the Azure CLI Bash script on a user defined Azure CLI version. If the user does not specify a version, latest CLI version is used.
Read more about various Azure CLI versions [here](https://github.com/Azure/azure-cli/releases).

- `azcliversion` – **Optional** Example: 2.0.72, Default: latest
- `inlineScript` – **Required** 

The definition of this GitHub Action is in [action.yml](https://github.com/Azure/CLI/blob/master/action.yml).  The action status is determined by the exit code returned by the script rather than StandardError stream. 

## Sample workflow 

### Dependencies on other GitHub Actions
* [Azure Login](https://github.com/Azure/login) – **Required** Login with your Azure credentials 
* [Checkout](https://github.com/actions/checkout) – **Optional** To execute the scripts present in your repository
### Workflow to execute an AZ CLI script of a specific CLI version
```
# File: .github/workflows/workflow.yml

on: [push]

name: AzureARMSample

env:
  LOCATION: westeurope
  RESOURCE_GROUP: rg-production-action

jobs:

  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    - name: Checkout
      uses: actions/checkout@v1
      
    - name: Azure CLI script
      uses: azure/CLI@v1
      with:
        azcliversion: 2.0.72
        inlineScript: |
          az group create --location $LOCATION --name $RESOURCE_GROUP
          az group deployment create --resource-group $RESOURCE_GROUP --template-file $GITHUB_WORKSPACE/azuredeploy.json
```

  * [GITHUB_WORKSPACE](https://help.github.com/en/github/automating-your-workflow-with-github-actions/virtual-environments-for-github-hosted-runners) is the environment variable provided by GitHub which represents the root of your repository.

### Configure Azure credentials as GitHub Secret:

To use any credentials like Azure Service Principal,add them as [secrets](https://help.github.com/en/articles/virtual-environments-for-github-actions#creating-and-using-secrets-encrypted-variables) in the GitHub repository and then use them in the workflow.

Follow the steps to configure the secret:
  * Define a new secret under your repository settings, Add secret menu
  * Store the output of the below [az cli](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest) command as the value of secret variable 'AZURE_CREDENTIALS'
```bash  

   az ad sp create-for-rbac --name "myApp" --role contributor \
                            --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} \
                            --sdk-auth
                            
  # Replace {subscription-id}, {resource-group} with the subscription, resource group details

  # The command should output a JSON object similar to this:

  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
  
```
  * Now in the workflow file in your branch: `.github/workflows/workflow.yml` replace the secret in Azure login action with your secret (Refer to the example above)


# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
