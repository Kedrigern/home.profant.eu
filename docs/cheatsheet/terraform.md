# Terraform 

Terraform resp. OpenTofu is a tool for IaC.

## Usage

```bash
terraform init    # download providers, initialize backend
terraform plan    # see what will be created/changed/deleted
terraform apply   # create/change/delete resources
terraform destroy # delete all resources created by this code
```

## Syntax

| **Element**   | Description                                                 | When to use                                                              |
|---------------|-------------------------------------------------------------|--------------------------------------------------------------------------|
| **resource**  | 	Creates and manages infrastructure.	                       | Almost always. It's the foundation.                                      |
| **data**	     | Loads information about existing infrastructure.            | When you need to use something not managed by your code.                 |
| **variable**	 | Defines inputs for your code (parameters).	                 | When you want your code to be configurable externally.                   |
| **locals**	   | Defines internal "helper" variables.	                       | To simplify code and avoid repetition.                                   |
| **output**	   | Defines outputs from your code.	                            | When you need to display an ID, IP address, or other resource attribute. |
| **provider**	 | Specifies which API (Azure, AWS, etc.) to communicate with. | Always. Defines the "language" Terraform speaks.                         |
| **module**	   | Encapsulates and enables code reuse.	                       | For all projects larger than a few files.                                |

## Structure

- every `tf` file in a directory is loaded
- usual files:
  - `main.tf` - main code
  - `variables.tf` - input variables
  - `outputs.tf` - output variables
  - `providers.tf` - provider configuration
  - `versions.tf` - provider and terraform version constraints
- modules are directories with their own `tf` files
- state file - keeps track of resources created by Terraform
  - can be local or remote (e.g. in Azure Storage, AWS S3, etc.)
  - remote state is recommended for teams

## Export existing resources from Azure

`aztfexport` can be used to export existing Azure resources to Terraform code. For example:
`aztfexport rg my-rg`. But it takes some cleanup afterward.

### Cleanup

```bash
tofu state rm <resource_type>.<resource_name> 
tofu state mv <resource_type>.<resource_name> <resource_type>.<new_resource_name> 
```

You can even move resource to each loop and rename them.

```bash
tofu state mv 'azurerm_eventhub.res-8' 'azurerm_eventhub.hubs["name"]'
```
