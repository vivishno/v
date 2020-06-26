# aml_configure

# GitHub Action for deploying/configuring Machine Learning Infrastructure to Azure

## Usage

The Configure action will deploy your infrastructure for [Azure Machine Learning](https://azure.microsoft.com/en-us/services/machine-learning/) on azure via arm template.If the resource being deployed already exists it will be modified according to the parameters provided.

Get started today with a [free Azure account](https://azure.com/free/open-source)!

This repository contains GitHub Action for deploying Machine Learning resources 

## Dependencies on other GitHub Actions
* [Checkout](https://github.com/actions/checkout) Checkout your Git repository content into GitHub Actions agent.



## Use of this GitHub Actions

This action is one in a starting point ins a series of actions that will be used in ML Ops process.
Using this action user will be able to deploy following resources to azure which will be required for further process to occur.
Execting this action using a workflow will deploy the following resources to azure-
* Machine Learning Workspace
* Function App having azure function


### Example workflow

```yaml
name: aml-train-deploy-workflow 
on:
  push:
    branches:
      - master
    # paths:
    #   - 'code/*'
jobs:
  train:
    runs-on: ubuntu-latest
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - name: Check Out Repository
      id: checkout_repository
      uses: actions/checkout@v2
        
    # Connect or Create the Azure Machine Learning Workspace
    - name: deploy all resources
      id: aml_configure
      uses: ./
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          armtemplate_file: "deploy.json"
          armtemplateparams_file: "deploy.params.json"
          resource_group: "AzureResourceGroupName"
          pattoken: ${{secrets.PAT_TOKEN}}

```

### Inputs

| Input | Required | Default | Description |
| ----- | -------- | ------- | ----------- |
| azure_credentials | x | - | Output of `az ad sp create-for-rbac --name <your-sp-name> --role contributor --scopes /subscriptions/<your-subscriptionId>/resourceGroups/<your-rg> --sdk-auth`. This should be stored in your secrets |
| armtemplate_file | "deploy.json" | - | We expect a JSON file in the `.cloud/.azure` folder in root of your repository specifying your model deployment details. If you have want to provide these details in a file other than "deploy.json" you need to provide this input in the action. |
| armtemplateparams_file | | deploy.params.json | - | We expect a JSON file in the `.cloud/.azure` folder in root of your repository specifying the parameters used by arm template file for deployment.The parameters can be configured by user accordingly. |
| resource_group |  | x | User needs to specify the azure resource group where the deployment of resources needs to be done.|
| pattoken |  | x | User needs to specify the github PAT token as value in github secrets under name `PAT_TOKEN` in your repository. This will be used by Function App to communicate to github |


#### azure_credentials ( Azure Credentials ) 

Azure credentials are required to connect to your Azure Machine Learning Workspace. These may have been created for an action you are already using in your repository, if so, you can skip the steps below.

Install the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) on your computer or use the Cloud CLI and execute the following command to generate the required credentials:

```sh
# Replace {service-principal-name}, {subscription-id} and {resource-group} with your Azure subscription id and resource group name and any name for your service principle
az ad sp create-for-rbac --name {service-principal-name} \
                         --role contributor \
                         --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} \
                         --sdk-auth
```

This will generate the following JSON output:

```sh
{
  "clientId": "<GUID>",
  "clientSecret": "<GUID>",
  "subscriptionId": "<GUID>",
  "tenantId": "<GUID>",
  (...)
}
```

Add this JSON output as [a secret](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#creating-encrypted-secrets) with the name `AZURE_CREDENTIALS` in your GitHub repository.


#### parameters_file (Parameters File)

The action tries to load a JSON file in the `.cloud/.azure` folder in your repository, which specifies details for the model deployment to your Azure Machine Learning Workspace. By default, the action expects a file with the name `deploy.json`. If your JSON file has a different name, you can specify it with this parameter. Note that none of these values are required and, in the absence, default sample file will be used.

A sample file can be found in this repository in the folder `.cloud/.azure`. The parameters file parameters are configurable and can be changed by user accordingly.

### Documentation of template file parameters

| Parameter                  | Description                                |
| ----------------------------- | ------------------------------------------ |
| `workspaceName`                        | Specifies the name of the Azure Machine Learning workspace.If the resource doesn't exist a new workspace will be created, else existing resource will be updated using the arm template file |
| `baseName`                  | Name used as base-template to name the resources to be deployed in Azure. |
| `OwnerName`         | Owner of this deployment, person to contact for question. |
| `GitHubBranch`  | Name of the branch containing azure function code. |
| `eventGridTopicPrefix`   | The name of the Event Grid custom topic. |
| `eventGridSubscriptionName`                 | The prefix of the Event Grid custom topic's subscription. |
| `FunctionName`        |name of azure function used|
| `subscriptionID` | azure subscription ID being used for deployment |
| `GitHubURL`           | The URL of GitHub (ending by .git) containing azure function code. |
| `funcProjectFolder`               | The name of folder containing the function code. |
| `repo_name`           | The name of repository containing template files.This is picked up from github environment parameter 'GITHUB_REPOSITORY' |
| `pat_token`                        | pat token to be used by the function app to communicate to github via repository dispatch. |


## Documentation of Azure Machine Learning GitHub Actions

The template uses the open source Azure certified Actions listed below. Click on the links and read the README files for more details.
- [aml-workspace](https://github.com/Azure/aml-workspace) - Connects to or creates a new workspace
- [aml-compute](https://github.com/Azure/aml-compute) - Connects to or creates a new compute target in Azure Machine Learning
- [aml-run](https://github.com/Azure/aml-run) - Submits a ScriptRun, an Estimator or a Pipeline to Azure Machine Learning
- [aml-registermodel](https://github.com/Azure/aml-registermodel) - Registers a model to Azure Machine Learning
- [aml-deploy](https://github.com/Azure/aml-deploy) - Deploys a model and creates an endpoint for the model

# What is MLOps?

<p align="center">
  <img src="docs/images/ml-lifecycle.png" alt="Azure Machine Learning Lifecycle" width="700"/>
</p>

MLOps empowers data scientists and machine learning engineers to bring together their knowledge and skills to simplify the process of going from model development to release/deployment. ML Ops enables you to track, version, test, certify and reuse assets in every part of the machine learning lifecycle and provides orchestration services to streamline managing this lifecycle. This allows practitioners to automate the end to end machine Learning lifecycle to frequently update models, test new models, and continuously roll out new ML models alongside your other applications and services.

This repository enables Data Scientists to focus on the training and deployment code of their machine learning project (`code` folder of this repository). Once new code is checked into the `code` folder of the master branch of this repository the GitHub workflow is triggered and open source Azure Machine Learning actions are used to automatically manage the training through to deployment phases.

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


