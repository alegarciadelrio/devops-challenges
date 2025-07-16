
# Code convention

- **spaces**: 2 spaces between each nesting level in Terraform code for better readability and maintainability.
- **default variable**: can't be empty {}

# Commands

- **terraform validate**: used to check and report errors within modules.
- **terraform state**: manage and manipulate the Terraform state in various ways.
- **terraform apply**: maximum of 10 concurrent resource operations. Controlled by parallelism configuration.

# Terraform backend

- **s3**: supported
- **consul**: supported
- **local**: supported
- **github**: NOT supported