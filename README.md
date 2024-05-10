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

> There must be two resources on Azure, for example;
>
> ![Resources on Azure Portal](https://github.com/polatengin/new-jersey/assets/118744/5f786fea-a77f-4527-89e1-e2da765f8a63)

- Create a new Azure Resource Manager Service Connection on Azure DevOps to connect the Azure Subscription.

> There must be the Service Connection on Azure DevOps, for example;
>
> ![Azure DevOps Service Connection](https://github.com/polatengin/new-jersey/assets/118744/4f9ecea7-7ac1-443c-aa1f-6cf46ec54a14)

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

> Sample terraform can be found in [./src/main.tf](./src/main.tf) file.

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

## Output

- The pipeline will run the Terraform commands on the Azure DevOps Pipeline.

- The Terraform Remote State files will be stored on the Azure Blob Storage.

## References

- [Terraform](https://www.terraform.io/)
- [Azure DevOps](https://dev.azure.com/)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)
- [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/)
- [Azure Storage Account](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-overview)
- [Azure Blob Storage](https://docs.microsoft.com/en-us/azure/storage/blobs/)
- [Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/)
- [Azure Resource Group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal)
- [Azure DevOps Pipeline](https://docs.microsoft.com/en-us/azure/devops/pipelines/)
- [Azure DevOps Variable Group](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups)
- [Azure DevOps Service Connection](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints)
- [Azure DevOps Pipeline Trigger](https://docs.microsoft.com/en-us/azure/devops/pipelines/build/triggers)
