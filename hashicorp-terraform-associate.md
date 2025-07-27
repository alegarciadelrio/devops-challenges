# 🛠️ HashiCorp Terraform Associate Certification Guide

## Table of Contents
- [Language](#language)
- [OS Binary](#os-binary)
- [Code Convention](#code-convention)
  - [Variables](#variables)
  - [Collections Types](#collections-types)
    - [Lists](#lists)
    - [Maps](#maps)
    - [Meta Arguments](#meta-arguments)
      - [for_each](#for-each)
    - [Functions](#functions)
  - [Block Types](#block-types)
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
- Can be used across major cloud providers and VM hypervisors

## OS Binary
- Linux
- Windows
- macOS
- Solaris
- FreeBSD

## Code Convention

### Variables
- Called expressions
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
- Enable most verbose level, providing the most detailed and specific logs `TF_LOG=TRACE`
- Variables name to avoid:
  - Reserved keywords: `source`,`version`, `providers`
  - Start with a number
  - Contain spaces or special characters
  - Only numbers
- `aws_instance.database["nomad"]` to refer `for_each` meta-argument

## Collections types
- list
- map
- object
- tuple

### Lists
- Lists are ordered collections of values
- Syntax: `["string1", "string2", "string3"]`

### Maps
- Allows for creating a collection of key-value pairs
- Syntax: `{ name = "John" age = 52 }`

### Meta arguments
- `depends_on`
- `count` or `for each`
- `provider`
- lifecycle
  - `create_before_destroy` for example change ami of ec2 instance
  - `prevent destroy` for example keep an s3 bucket
  - `replace_triggered_by` replaces a resouce when an item change
  - `ignore_changes` provide a list of attributes that should not trigger an update

#### for_each
- When using a for_each argument in a resource block `each.value.ip`

### Functions
- min, max, format, join, trim, and length

### Block types
- provider 
- terraform 
- output 
- data 
- resource

### Terraform Block
- Only constants are allowed
- Input variables not allowed

### Data Block
- Data blocks are used to retrieve data from external sources
- Filters can be applied to filter by name or values
- Reference data from other resources using syntax like `data.aws_ami.example.id`
- Called `data source` as well

### Provider Block
- A plugin that Terraform uses to interact with a cloud provider or other infrastructure service
- Multiple providers can be configured using the `alias` parameter, great for `multi region`
- `required_providers` acts as a traffic controller for infrastructure tools
- AWS provider minimum version: `~> 5.36.0`
- Provider dependencies are created when:
  - `required_providers` is used (Optionally version)
  - An existing resource is in the current state
  - A resource is referenced in the configuration
- Good practice, declare the required version of a provider
- Not a required block

### Output Block
- Adding an output block to a module exposes the ID or value as an output variable that can be retrieved using `module.name.name_id`

### Dynamic Block
- Dynamic blocks allow you to create a list of resources based on a variable
- Produces nested configuration blocks instead of a complex typed value

### Locals Block
- `locals {}` block type is used to define local variables within a Terraform module

## Commands

### Terraform Init
- First command to run before planning
- Initializes the working directory
- Automatically download community providers
- Initializes the backend configuration
- Caches the source code locally for referenced modules
- Download providers plugins
- Download modules from the registry
- `terraform init -upgrade` to upgrade plugins and modules to the latest version
- Terraform would use the existing module already downloaded if you remove the version attribute

### Terraform Validate
- Checks for errors in the configuration files
- Variable is being used in a resource block but has not been declared

### Terraform fmt
- Formats the configuration files
- `-recursive` flag to also process files in subdirectories

### Terraform Get
- Downloads modules from the registry

### Terraform Plan
- Preview changes to infrastructure without applying them
- Shows what changes will be made
- `terraform plan -refresh-only` command is used in Terraform to update the state of your infrastructure in memory without making any actual changes to the infrastructure.
- `terraform plan -out=bryan` used to run a dry-run and save it on a file
- `terraform plan -destroy` used to run a plan to destroy the infrastructure

### Terraform Apply
- Applies the changes to the infrastructure
- Maximum of 10 concurrent resource operations. Controlled by parallelism configuration.
- `terraform apply -replace=aws_instance.web` to mark the resource for replacement
- `terraform apply -refresh-only` is used to reconcile the state with the real-world infrastructure by refreshing the state without making any changes
- `terraform apply -destroy`

### Terraform State
- Manage and manipulate Terraform state
- State files track infrastructure resources and compare with desired state
- Commands:
  - `terraform state rm`: Remove resource from state without destroying it
  - `terraform state list`: List all Terraform-managed resources
  - `terraform state mv`: update the state to match the current deployment. Terraform would not touch the actual resource that is deployed, but it would simply attach the existing object to the new address in Terraform.
- state locking, ensure the state does not become corrupt with the remote state

### Terraform Show
- Inspect the state file

### Terraform force-unlock 
- This command is specifically designed to force unlock the state file and allow modifications to be made.

### Terraform Import
- Import existing infrastructure into Terraform state
- Before importing, update the Terraform configuration file to include the new resources that match the resources you want to import
- The terraform import command does not automatically update the referenced Terraform resource block after importing the resource
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
- `terraform destroy -auto-approve` to destroy resources without confirmation
- `terraform destroy -target=azurerm_resource_group.production` to destroy particular resource

### Terraform Output
- Used to display the values of outputs defined in the Terraform configuration

## Terraform Backend
- Defines where state data is stored and how operations are performed, but it does not provide an abstraction above the upstream API
- Requires authentication
- Supported backends:
    - S3
    - Consul
    - Local
    - Remote (Can support or not state locking, HCP Terraform)
    - Kubernetes
- Unsupported backend:
    - GitHub
- Ways to pass secrets to the terraform backend:
    - HashiCorp Vault
    - Interactevely through command line
    - `-backend-config=PATH` to specify a separate file for backend configuration
- Has the following files:
    - state file
    - config file
- Dynamic backend in Terraform is used to configure different backend configurations based on conditions or variables
- Backend block can't receive input var, resource ref and data sources
- When changes are made, we must re run `terraform init`
- `terraform init -migrate-state` to migrate the state file when you change backend directory

## Terraform modules
- Store remotely or locally
- Code reuse
- Enable versioning
- Stored in `.terraform/modules`

## Terraform Cloud
- Cloud migration configures workspace to use same version as Terraform binary
- Supported version control systems:
  - GitHub.com
  - GitHub Enterprise
  - Azure DevOps
  - Bitbucket Cloud
- Sentinel and OPA with Terraform Cloud:
  - Policy enforcement as a framework
  - Governance, compliance, naming conventions, approved machine images, etc
  - Security
- Sentinel policies run after the plan, run tasks, cost estimation phases but before the apply phase

## Dependencies
- Explicit dependencies: Use `depends_on` argument in resource blocks to declare dependencies
- Takes a list of resource names to specify dependencies
- Implicit dependencies are not explicitly declared in the configuration but are automatically detected by Terraform

## Files
- Workspace states are stored in `terraform.tfstate.d/<workspace_name>` directory
- Downloaded provider plugins are stored in the `.terraform/providers` directory
- Terraform community store `state` files on the current working directory
- No mandatory `main.tf`
- `.tfstate` file mandatory and not encrypted
- `.tfstate.backup` file come up after a destoy and not encrypted
- `.terraform.lock.info` file come up on action over state file, it is to lock the state

## Terraform Public Registry
- Modules must be published on GitHub with public access
- Version tags must follow x.y.z format (optional 'v' prefix)
- Module names must follow format: `terraform-<PROVIDER>-<NAME>`
- Standard module structure is required for registry inspection and documentation generation
- Specify TPR using: `source = "terraform-vault-aws-tgw/hcp"`

## Private Registry
- Publish custom modules in a private registry
- Version tags must follow `x.y.z` format (optional 'v' prefix)

## HCP Terraform
- Needs API key to manage
- Workspace will be associated with a single repository
- Workspaces can be created/switched directly within the HCP Terraform web interface without the need to use the CLI
- Uses workspaces to manage collections of infrastructure, while the CLI manages collections of infrastructure resources within a persistent working directory
- Private registry
- Remote runs
- VCS connection

## HCP Terraform agents
- Bridge between HCP and the infraestructure (Permits plan and apply)

## HashiCorp Vault
- Best way to read and write secrets is using a vault provider
- Access `access_key = data.vault_aws_access_credentials.creds.access_key`

## Security
- Ways to provide credentials to terraform
  - By the provider block hardcoding
  - By environment variables
  - By integrated services IAM or Azure Managment Service Identity. 