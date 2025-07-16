# Language
- Terraform is an immutable, declarative, IasC provisioning language based on Hashicorp Configuration Language, or optionally JSON.

# Code convention

- **spaces**: 2 spaces between each nesting level in Terraform code for better readability and maintainability.
- **default variable**: can't be empty {}

# Code block data

- **data blocks**: used to retrieve data from external sources.
- **filter**: can filter by name or values.
- **data.aws_ami.exanple.id**: way to refer the data retrieved from other resources.

# Code block provider

- **multiple providers**: use alias parameter. Allows differentiate between the different configurations of the same provider type.

# Commands

- **terraform validate**: used to check and report errors within modules.
- **terraform apply**: maximum of 10 concurrent resource operations. Controlled by parallelism configuration.

## Terraform plan

- **terraform plan**: preview the changes that will be made to the infrastructure without actually applying them

## Terraform state

- **terraform state**: manage and manipulate the Terraform state in various ways.
- **state files** needed to track the current state of infrastructure resources and compare it to the desired state declared in configuration files.

# Terraform backend

- **s3**: supported
- **consul**: supported
- **local**: supported
- **remote**: supported
- **github**: NOT supported

# Cloud migration

- **Terraform Cloud**: configures the workspace to use the same version as the Terraform binary you used when migrating.
