# Quick Reference: Terraform Variable Types & Precedence

## 1. Variable Types at a Glance 🔍

### Basic Types
| Type | Use Case | Example | Note |
|------|-----------|---------|------|
| `string` | Text values | `"t2.micro"` | Most common type, used for names, IDs |
| `number` | Numeric values | `42` | For counts, ports, sizes |
| `bool` | True/False | `true` | For enabling/disabling features |
| `list` | Ordered collection | `["us-east-1a", "us-east-1b"]` | For multiple similar items |
| `map` | Key-value pairs | `{"env" = "prod"}` | For lookups and attributes |

### Complex Types
| Type | Use Case | Example | Note |
|------|-----------|---------|------|
| `object` | Structured data | `{name = string, count = number}` | For complex configurations |
| `tuple` | Mixed type list | `[string, number, bool]` | When list elements have different types |
| `set` | Unique values | `["web", "app"]` | Prevents duplicates |

### Quick Type Examples
```hcl
# String with validation
variable "env" {
  type = string
  validation {
    condition = contains(["dev", "prod"], var.env)
    error_message = "Must be dev or prod."
  }
}

# Number with range check
variable "port" {
  type = number
  validation {
    condition = var.port > 0 && var.port < 65536
    error_message = "Port must be valid."
  }
}

# Complex object
variable "server" {
  type = object({
    name  = string
    ports = list(number)
    tags  = map(string)
  })
}
```

## 2. Variable Precedence Cheat Sheet 📊

### Precedence Order (Highest → Lowest)
1. Command-line flags (`-var` or `-var-file`)
2. `*.auto.tfvars` or `*.auto.tfvars.json` files
3. `terraform.tfvars` or `terraform.tfvars.json`
4. Environment variables (`TF_VAR_*`)
5. Default values in variable declarations

### Quick Examples of Each Level

```bash
# Level 1: Command-line (Highest)
terraform apply -var="env=prod" -var-file="custom.tfvars"

# Level 2: *.auto.tfvars
prod.auto.tfvars:
env = "production"

# Level 3: terraform.tfvars
terraform.tfvars:
env = "development"

# Level 4: Environment Variables
export TF_VAR_env="staging"

# Level 5: Default Values (Lowest)
variable "env" {
  default = "dev"
}
```

### Precedence in Action
```hcl
# Scenario: Same variable defined at multiple levels

# variables.tf (Level 5)
variable "instance_type" {
  type    = string
  default = "t2.nano"  # Lowest precedence
}

# terraform.tfvars (Level 3)
instance_type = "t2.micro"

# prod.auto.tfvars (Level 2)
instance_type = "t2.small"

# Command Line (Level 1)
# terraform apply -var="instance_type=t2.medium"

# Final value will be: "t2.medium" (Command line wins)
```

## 3. Quick Tips 💡

### Variable Types
- Start with basic types when possible
- Use `object` for complex configurations
- Use `map` for lookups and dynamic values
- Use `list` for ordered collections
- Use `set` when uniqueness matters

### Precedence
- Use command-line for temporary overrides
- Use `.auto.tfvars` for environment-specific values
- Use `terraform.tfvars` for project defaults
- Use environment variables for sensitive data
- Use default values as fallbacks only

### Common Gotchas ⚠️
1. Type Mixing:
```hcl
# Won't work
variable "ports" {
  type    = list(number)
  default = [80, "443"]  # Error: string in number list
}
```

2. Precedence Override Surprise:
```hcl
# terraform.tfvars
instance_type = "t2.micro"

# dev.auto.tfvars
# This will override terraform.tfvars due to higher precedence
instance_type = "t2.small"
```

3. Environment Variable Format:
```bash
# Correct
export TF_VAR_instance_type="t2.micro"

# Wrong (missing TF_VAR_ prefix)
export instance_type="t2.micro"
```
