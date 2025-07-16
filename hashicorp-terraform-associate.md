# Language
- Terraform is an immutable, declarative, IasC provisioning language based on Hashicorp Configuration Language, or optionally JSON.

## Code convention

- **spaces**: 2 spaces between each nesting level in Terraform code for better readability and maintainability.
- **default variable**: can't be empty {}

### Code block data

- **data blocks**: used to retrieve data from external sources.
- **filter**: can filter by name or values.
- **data.aws_ami.exanple.id**: way to refer the data retrieved from other resources.

### Code block provider

- **multiple providers**: use alias parameter. Allows differentiate between the different configurations of the same provider type.

### Output block

Adding an output block to a module allows the id or value to be exposed as an output variable. This output variable can then be retrieved using module.name.name_id.

## Commands

- **terraform validate**: used to check and report errors within modules.
- **terraform apply**: maximum of 10 concurrent resource operations. Controlled by parallelism configuration.

### Terraform init

- **terraform init**: first command before plan.

### Terraform plan

- **terraform plan**: preview the changes that will be made to the infrastructure without actually applying them.

### Terraform state

- **terraform state**: manage and manipulate the Terraform state in various ways.
- **state files**: needed to track the current state of infrastructure resources and compare it to the desired state declared in configuration files.
- **terraform state rm**: permits remove a resource from the state, to prevent detroy it when you run `terraform destroy`.
- **terraform state list**: see a list of all Terraform-managed resources.

### Terraform destroy

Will prompt for user confirmation.

## Terraform backend

- **s3**: supported
- **consul**: supported
- **local**: supported
- **remote**: supported
- **github**: NOT supported

## Cloud migration

- **Terraform Cloud**: configures the workspace to use the same version as the Terraform binary you used when migrating.

## Dependencies

- Declare a resource dependency, you can use the depends_on argument in a resource block. The depends_on argument takes a list of resource names and specifies that the resource block in which it is declared depends on those resources.

## Files

- Terraform stores the workspace states in a directory called terraform.tfstate.d/<workspace_name>

## Terraform Public Registry

- **Github**: module should published on github. Public access mode.
- **Version**: release tag names to follow the x.y.z format and optionally be prefixed with a 'v' is valid.
- **Module name**: repositories must use this three-part name format, terraform-<PROVIDER>-<NAME>
- **Standard module structure**: allows the registry to inspect your module and generate documentation.
