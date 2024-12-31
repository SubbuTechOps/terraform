# Terraform Variable Precedence Guide

## Order of Precedence (Lowest to Highest)

1. **Default Variable Values** (variables.tf)
   - Lowest precedence
   - Used when no other values are provided
   - Good for safe defaults
   ```hcl
   variable "environment" {
     default = "dev"
   }
   ```

2. **Environment Variables** (TF_VAR_*)
   - Prefix variables with `TF_VAR_`
   - Useful for sensitive values
   - Example: `export TF_VAR_region="us-west-2"`

3. **terraform.tfvars**
   - Automatically loaded
   - Project-specific defaults
   - Don't commit sensitive values

4. ***.auto.tfvars** (alphabetical order)
   - Automatically loaded
   - Good for environment-specific values
   - Example: `production.auto.tfvars`

5. **Named .tfvars files** (-var-file flag)
   - Explicitly loaded via CLI
   - `terraform plan -var-file="prod.tfvars"`

6. **Command Line Flags** (-var)
   - Highest precedence
   - Overrides all other values
   - Example: `terraform plan -var="region=us-east-1"`

## Best Practices

1. **Sensitive Values**
   - Use environment variables
   - Never commit to version control
   - Example: `export TF_VAR_db_password="secret"`

2. **Environment Management**
   - Use separate .tfvars files per environment
   - Store in `environments/` directory
   - Example: `environments/prod.tfvars`

3. **Default Values**
   - Always provide safe defaults
   - Document expected values
   - Use validation blocks

## Common Interview Questions

Q1: How would you handle different environments in Terraform?
```
A: Best practices include:
- Separate .tfvars files per environment
- Store in environments/ directory
- Use workspaces for isolation
- Example structure:
  environments/
  ├── dev.tfvars
  ├── staging.tfvars
  └── prod.tfvars
```

Q2: How do you manage sensitive variables in Terraform?
```
A: Multiple approaches:
1. Environment variables (TF_VAR_*)
2. Encrypted .tfvars files
3. Secret management services
4. Never commit sensitive values
```

Q3: What's the difference between terraform.tfvars and *.auto.tfvars?
```
A: 
- terraform.tfvars: Single file, automatically loaded
- *.auto.tfvars: Multiple files, loaded alphabetically
- Both are automatic, but auto.tfvars allows multiple files
```

Q4: How would you override a variable for a specific run?
```
A: Use command line flags:
terraform plan \
  -var-file="environments/prod.tfvars" \
  -var="instance_type=t2.large"
```

## Tips and Tricks

1. **Variable Validation**
```hcl
variable "environment" {
  type = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

2. **Default Tags**
```hcl
provider "aws" {
  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "terraform"
    }
  }
}
```

3. **Variable Debugging**
```bash
# Print all variables
terraform console
> var.environment

# Check resolved values
terraform plan -out=plan.tfplan
terraform show plan.tfplan
```

4. **Environment Variables**
```bash
# Set multiple variables
export TF_VAR_region="us-west-2"
export TF_VAR_environment="prod"

# Unset variables
unset TF_VAR_region
```

## Common Gotchas

1. **Variable Precedence Overrides**
   - Higher precedence completely replaces lower
   - No merging of maps/lists across sources

2. **Environment Variables Format**
   - Must use TF_VAR_ prefix
   - Underscores in names, not dashes

3. **Auto-loading Limitations**
   - Only terraform.tfvars and *.auto.tfvars
   - Other files need explicit -var-file
