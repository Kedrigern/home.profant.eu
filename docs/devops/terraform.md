---
title: "Terraform / OpenTofu"
icon: simple/terraform
tags:
  - devops
  - iac
---
Terraform resp. OpenTofu is a tool for IaC.

## Usage

```bash
tofu init    # download providers, initialize backend
tofu plan    # see what will be created/changed/deleted
tofu apply   # create/change/delete resources
tofu destroy # delete all resources created by this code
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

## Principles

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

## Architecture

```
├── environmnets
│   ├── dev
│   └── prd
└── modules
    ├── identity
    ├── keyvault
    ├── my_project
    ├── servicebus
    └── storage
```

Where:

- `environments` - contains environment-specific code (e.g. dev, prd) and `statefile` if local
- every environment call main module `my_project` and pass environment-specific parameters
- module `my_project` calls other modules like `identity`, `keyvault`, etc.

## Import resource

```
tofu import <resource_type>.<resource_name> <resource_id>
tofu import azurerm_resource_group.rg /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/my-rg
```

### Export existing resources from Azure

`aztfexport` can be used to export existing Azure resources to Terraform code. For example:
`aztfexport rg my-rg`. But it takes some cleanup afterward.

### Cleanup

```
tofu state rm <resource_type>.<resource_name> 
tofu state mv <resource_type>.<resource_name> <resource_type>.<new_resource_name> 
```

You can even move resource to each loop and rename them.

```
tofu state mv 'azurerm_eventhub.res-8' 'azurerm_eventhub.hubs["name"]'
```
