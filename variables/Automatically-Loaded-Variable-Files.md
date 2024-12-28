# Terraform Automatically Loaded Variable Files

## File Naming Conventions

Terraform automatically loads variable definitions from the following files, in this order:

1. `terraform.tfvars` or `terraform.tfvars.json`
   - Standard default variable files
   - Always loaded if present
   - Good for common or default values

2. `*.auto.tfvars` or `*.auto.tfvars.json`
   - Any files with names ending in `.auto.tfvars` or `.auto.tfvars.json`
   - Automatically loaded in alphabetical order
   - Perfect for environment-specific configurations

## Best Practices

1. Name Usage:
   - Use `terraform.tfvars` for common variables shared across environments
   - Use `*.auto.tfvars` for environment-specific variables (e.g., `production.auto.tfvars`, `staging.auto.tfvars`)

2. File Organization:
```plaintext
project/
├── main.tf
├── variables.tf
├── terraform.tfvars       # Common variables
├── dev.auto.tfvars       # Development environment
├── staging.auto.tfvars   # Staging environment
└── prod.auto.tfvars      # Production environment
```

## Example Usage

```hcl
# terraform.tfvars (common variables)
project_name = "myapp"
region = "us-west-2"

# production.auto.tfvars
environment = "production"
instance_type = "t2.large"
min_capacity = 3

# development.auto.tfvars
environment = "development"
instance_type = "t2.micro"
min_capacity = 1
```

## Important Notes

1. Auto-loading is based on file extension
2. Files are processed in alphabetical order for same-type files
3. No need to explicitly specify these files on the command line
4. Values in `*.auto.tfvars` override those in `terraform.tfvars`
5. Both formats (`.tfvars` and `.tfvars.json`) are supported
