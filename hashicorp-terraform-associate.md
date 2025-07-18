# 🛠️ HashiCorp Terraform Associate Certification Guide

## Table of Contents
- [Language](#language)
- [OS Binary](#os-binary)
- [Code Convention](#code-convention)
  - [Variables](#variables)
  - [Block Types](#block-types)
  - [Code Block Data](#code-block-data)
  - [Code Block Provider](#code-block-provider)
  - [Output Block](#output-block)
- [Commands](#commands)
  - [Terraform Init](#terraform-init)
  - [Terraform Validate](#terraform-validate)
  - [Terraform Get](#terraform-get)
  - [Terraform Refresh](#terraform-refresh)
  - [Terraform Plan](#terraform-plan)
  - [Terraform Apply](#terraform-apply)
  - [Terraform State](#terraform-state)
  - [Terraform Force-unlock](#terraform-force-unlock)
  - [Terraform Import](#terraform-import)
  - [Terraform Workspace](#terraform-workspace)
  - [Terraform Destroy](#terraform-destroy)
- [Terraform Backend](#terraform-backend)
- [Terraform Cloud](#terraform-cloud)
  - [Private Registry](#private-registry)
  - [Sentinel and OPA](#sentinel-and-opa)
- [Dependencies](#dependencies)
- [Files](#files)
- [Terraform Public Registry](#terraform-public-registry)

## Language
- Terraform is an immutable, declarative, IaC provisioning language based on HashiCorp Configuration Language (HCL) or optionally JSON
- HCL (HashiCorp Configuration Language)
- JSON
- Secrets: Terraform does not provide the ability to mask secrets in the Terraform plan and state files regardless of what provider you are using

## OS Binary
- Linux
- Windows
- macOS
- Solaris
- FreeBSD

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
- Types: string, number, bool, list (or tuple), map (or object)
- Enable detail logging: `TF_LOG=DEBUG`
- Variables name avoid:
  - Reserved keywords: `source`,`version`, `providers`
  - start with a number
  - contain spaces or special characters
  - only numbers

### Block types
- provider 
- terraform 
- output 
- data 
- resource

### Code Block Data
- Data blocks are used to retrieve data from external sources
- Filters can be applied to filter by name or values
- Reference data from other resources using syntax like `data.aws_ami.example.id`

### Code Block Provider
- Multiple providers can be configured using the alias parameter
- `required_providers` acts as a traffic controller for infrastructure tools
- AWS provider minimum version: ~> 5.36.0
- Provider dependencies are created when:
  - `required_providers` is used (Optionally version)
  - An existing resource is in the current state
  - A resource is referenced in the configuration
- Good practice, declare the required version of a provider

### Output Block
- Adding an output block to a module exposes the ID or value as an output variable that can be retrieved using `module.name.name_id`

## Commands

### Terraform Init
- First command to run before planning
- Initializes the working directory
- Automatically download community providers. 

### Terraform Validate
- Checks for errors in the configuration files

### Terraform Get
- Downloads modules from the registry

### Terraform Refresh
- Refreshes the state of the infrastructure

### Terraform Plan
- Preview changes to infrastructure without applying them
- Shows what changes will be made
- `terraform plan -refresh-only` command is used in Terraform to update the state of your infrastructure in memory without making any actual changes to the infrastructure.
- `terraform plan -out=bryan` used to run a dry-run and save it on a file

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
- state locking, ensure the state does not become corrupt with the remote state

### Terraform force-unlock 
- This command is specifically designed to force unlock the state file and allow modifications to be made.

### Terraform Import
- Import existing infrastructure into Terraform state
- Syntax: `terraform import aws_instance.web i-12345678`

### Terraform Workspace
- Terraform workspace is a way to manage multiple environments
- Uses its own file for each workspace.
- Syntax: `terraform workspace new <workspace_name>`
- Syntax: `terraform workspace select <workspace_name>`
- Syntax: `terraform workspace list`
- Terraform workspaces Community and HCP are differents

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

Ways to pass secrets to the terraform backend:
- HashiCorp Vault
- Interactevely through command line
- `-backend-config=PATH` to specify a separate file for backend configuration

Has the following files:
- state file
- config file

## Terraform Cloud
- Cloud migration configures workspace to use same version as Terraform binary
- Supported version control systems:
  - GitHub.com
  - GitHub Enterprise
  - Azure DevOps
  - Bitbucket Cloud
- **Private Registry**: publish custom modules in a private registry.
- Sentinel and OPA with Terraform Cloud:
  - Policy enforcement
  - Governance, compliance, naming conventions, approved machine images, etc
  - Security


## Dependencies
- Explicit dependencies: Use `depends_on` argument in resource blocks to declare dependencies
- Takes a list of resource names to specify dependencies
- Implicit dependencies are not explicitly declared in the configuration but are automatically detected by Terraform.

## Files
- Workspace states are stored in `terraform.tfstate.d/<workspace_name>` directory
- Downloaded provider plugins are stored in the `.terraform/providers` directory

## Terraform Public Registry
- Modules must be published on GitHub with public access
- Version tags must follow x.y.z format (optional 'v' prefix)
- Module names must follow format: `terraform-<PROVIDER>-<NAME>`
- Standard module structure is required for registry inspection and documentation generation
- Specify TPR using: `source = "terraform-vault-aws-tgw/hcp"`

## HashiCorp Vault
- Best way to read and write secrets is using a vault provider.
