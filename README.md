# New Jersey

Purpose of this project is to demonstrate how to run terraform commands on an Azure DevOps Pipeline and how to setup Remote State Storage that configured on Azure Blob Storage.

## Prerequisites

- Azure Subscription
- Azure DevOps Account
- Azure Resource Group
- Azure Storage Account
- Azure Blob Container
- Azure Key Vault

## Steps

- Create a new project on Azure DevOps with a repository.

- Create a new Resource Group on the Azure Subscription.

```bash
az group create --name <resource-group-name> --location <location>
```

- Create a new Storage Account on the Azure Subscription.

```bash
az storage account create --name <storage-account-name> --resource-group <resource-group-name> --location <location> --sku Standard_LRS
```

- Create a new Blob Container on the Azure Storage Account to store the Terraform Remote State files (e.g. `tfstate`).

```bash
az storage container create --name <container-name> --account-name <storage-account-name> --account-key <storage-account-key>
```

- Create a new Key Vault on the Azure Subscription.

```bash
az keyvault create --name <key-vault-name> --resource-group <resource-group-name> --location <location>
```

- Create a new secret on the Azure Key Vault to store the Storage Account Access Key.

```bash
az keyvault secret set --vault-name <key-vault-name> --name <secret-name> --value <storage-account-key>
```

- Create a new Azure Resource Manager Service Connection on Azure DevOps to connect the Azure Subscription.

- Create a new Variable Group on Azure DevOps that has a link to the Azure Key Vault.

- Add the secret as a variable to the Variable Group.

- Configure the Terraform file to use the Azure Storage Account as a Remote State Storage.

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>3.0"
    }
  }
  backend "azurerm" {
    resource_group_name  = "<container-name>"
    storage_account_name = "<storage-account-name>"
    container_name       = "<container-name>"
    key                  = "<state-file-name>"
  }
}

provider "azurerm" {
  features {}
}
```

- Create a new Pipeline on Azure DevOps.

- Attach the Variable Group to the Pipeline.

```yaml
variables:
- group: pipeline-variables
```

- Add a script task to the pipeline to run terraform plan command

```yaml
- script: |
    terraform init
    terraform plan
  displayName: 'Terraform Plan'
  workingDirectory: '$(System.DefaultWorkingDirectory)/src'
  env:
    ARM_ACCESS_KEY: "$(REMOTE-STATE-STORAGE-CONNECTION-STRING)"
```

- Add a script task to the pipeline to run terraform apply command

```yaml
- script: |
    terraform apply -auto-approve
  displayName: 'Terraform Apply'
  workingDirectory: '$(System.DefaultWorkingDirectory)/src'
  env:
    ARM_ACCESS_KEY: "$(REMOTE-STATE-STORAGE-CONNECTION-STRING)"
```

- Commit the changes to the repository.

- Run the pipeline.
