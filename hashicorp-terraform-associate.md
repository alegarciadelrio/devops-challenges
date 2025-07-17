# HashiCorp Terraform Associate Certification Guide

## Table of Contents
- [Language](#language)
- [Code Convention](#code-convention)
  - [Code Block Data](#code-block-data)
  - [Code Block Provider](#code-block-provider)
  - [Output Block](#output-block)
- [Commands](#commands)
  - [Terraform Init](#terraform-init)
  - [Terraform Plan](#terraform-plan)
  - [Terraform State](#terraform-state)
  - [Terraform Destroy](#terraform-destroy)
- [Terraform Backend](#terraform-backend)
- [Terraform Cloud](#terraform-cloud)
- [Dependencies](#dependencies)
- [Files](#files)
- [Terraform Public Registry](#terraform-public-registry)

## Language
- Terraform is an immutable, declarative, IaC provisioning language based on HashiCorp Configuration Language (HCL) or optionally JSON
- HCL (HashiCorp Configuration Language)
- JSON
- Secrets: Terraform does not provide the ability to mask secrets in the Terraform plan and state files regardless of what provider you are using


## Code Convention

### Variables
- In your Terraform code: `variable "bucket_name" { ... }`
- In terraform.tfvars: `bucket_name = "my-bucket"`
- Or via environment variable: `TF_VAR_bucket_name=my-bucket`
- Align the equals signs
  ```hcl
    ami           = "abc123" 
    instance_type = "t2.micro"
  ```

### Code Block Data
- Data blocks are used to retrieve data from external sources
- Filters can be applied to filter by name or values
- Reference data from other resources using syntax like `data.aws_ami.example.id`

### Code Block Provider
- Multiple providers can be configured using the alias parameter
- `required_providers` acts as a traffic controller for infrastructure tools
- AWS provider minimum version: ~> 5.36.0

### Output Block
- Adding an output block to a module exposes the ID or value as an output variable that can be retrieved using `module.name.name_id`

## Commands

### Terraform Init
- First command to run before planning
- Initializes the working directory

### Terraform Validate
- Checks for errors in the configuration files

### Terraform Get
- Downloads modules from the registry

### Terraform Refresh
- Refreshes the state of the infrastructure

### Terraform Plan
- Preview changes to infrastructure without applying them
- Shows what changes will be made

### Terraform Apply
- Applies the changes to the infrastructure
- Maximum of 10 concurrent resource operations. Controlled by parallelism configuration.
- `terraform apply -replace=aws_instance.web` to mark the resource for replacement

### Terraform State
- Manage and manipulate Terraform state
- State files track infrastructure resources and compare with desired state
- Commands:
  - `terraform state rm`: Remove resource from state without destroying it
  - `terraform state list`: List all Terraform-managed resources
  - `terraform state mv`: update the state to match the current deployment. Terraform would not touch the actual resource that is deployed, but it would simply attach the existing object to the new address in Terraform.

### Terraform Import
- Import existing infrastructure into Terraform state
- Syntax: `terraform import aws_instance.web i-12345678`

### Terraform Destroy
- Prompts for user confirmation before destroying resources

## Terraform Backend
Supported backends:
- S3
- Consul
- Local
- Remote
Unsupported backend:
- GitHub

## Terraform Cloud
- Cloud migration configures workspace to use same version as Terraform binary
- Supported version control systems:
  - GitHub.com
  - GitHub Enterprise
  - Azure DevOps
  - Bitbucket Cloud
- **Private Registry**: publish custom modules in a private registry.

## Dependencies
- Use `depends_on` argument in resource blocks to declare dependencies
- Takes a list of resource names to specify dependencies

## Files
- Workspace states are stored in `terraform.tfstate.d/<workspace_name>` directory
- Downloaded provider plugins are stored in the `.terraform/providers` directory

## Terraform Public Registry
- Modules must be published on GitHub with public access
- Version tags must follow x.y.z format (optional 'v' prefix)
- Module names must follow format: `terraform-<PROVIDER>-<NAME>`
- Standard module structure is required for registry inspection and documentation generation
- Specify TPR using: `source = "terraform-vault-aws-tgw/hcp"`