# Terraform Variables: Essential Interview Points

> 💡 Quick Note: Understanding variables is crucial as they make your Terraform code reusable and maintainable.

## 1. Variable Types & Declarations
> 🔑 Key Point: Terraform supports both basic and complex data types. Choose the right type for your use case.

### Basic Types
```hcl
# String: For text values like names, regions, etc.
variable "environment" {
  type    = string
  default = "development"
}

# Number: For counts, sizes, ports
variable "instance_count" {
  type    = number
  default = 2
}

# Boolean: For yes/no, true/false decisions
variable "enable_monitoring" {
  type    = bool
  default = true
}

# List: For ordered collections of values
variable "availability_zones" {
  type    = list(string)
  default = ["us-west-2a", "us-west-2b"]
}

# Map: For key-value pairs, like tags or configurations
variable "instance_types" {
  type = map(string)
  default = {
    dev  = "t2.micro"
    prod = "t2.large"
  }
}
```
> 📝 Note: Basic types are most commonly used and should be your first choice unless you need complex structures.

### Complex Types
```hcl
# Object: For structured data with different types
variable "vpc_settings" {
  type = object({
    cidr_block     = string
    dns_support    = bool
    instance_count = number
  })
}

# Tuple: For fixed-size lists with different types
variable "port_config" {
  type    = tuple([string, number, bool])
  default = ["tcp", 80, true]
}

# Set: For unique values only
variable "unique_tags" {
  type    = set(string)
  default = ["app", "web", "public"]
}
```
> ⚠️ Note: Complex types are powerful but make your code harder to maintain. Use them only when necessary.

## 2. Variable Precedence (Highest to Lowest)
> 🔑 Key Point: Understanding precedence is crucial for debugging variable values.

1. Command line flags (-var and -var-file)
   > Used for temporary overrides
2. *.auto.tfvars files
   > For environment-specific values
3. terraform.tfvars
   > For default project values
4. Environment variables (TF_VAR_*)
   > For sensitive data and CI/CD
5. Default values in variable declarations
   > Fallback values

## 3. Important Interview Questions & Answers

### Q1: What are the different ways to pass variables in Terraform?
> 🎯 Focus: There are multiple ways to set variables, each with its own use case.

```bash
# Command line: For temporary changes
terraform apply -var="instance_type=t2.micro"

# Variable files: For environment configs
instance_type = "t2.micro"  # in terraform.tfvars

# Environment variables: For sensitive data
export TF_VAR_instance_type="t2.micro"

# Default values: For fallback options
variable "instance_type" {
  default = "t2.micro"
}
```

### Q2: How do you handle sensitive variables?
> 🔒 Security Note: Never store sensitive values in version control.

```hcl
variable "db_password" {
  type      = string
  sensitive = true  # Masks value in logs
}
```

### Q3: Explain variable validation
> ✅ Validation ensures input values meet your requirements.

```hcl
variable "environment" {
  type = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}
```

## 4. Advanced Concepts

### Dynamic Blocks
> 🔄 Use for repeatable nested blocks.
```hcl
# Creates multiple ingress rules from a single variable
variable "ingress_rules" {
  type = list(object({
    port        = number
    protocol    = string
    cidr_blocks = list(string)
  }))
}
```

### Conditional Usage
> 🔍 For environment-specific configurations.
```hcl
# Different values based on environment
locals {
  instance_count = var.environment == "prod" ? 3 : 1
}
```

## 5. Best Practices
> 📌 Follow these for maintainable code.

1. Variable Naming
   - Use snake_case
   - Be descriptive
   - Stay consistent

2. Documentation
   - Always add descriptions
   - Include examples
   - Document requirements

3. Organization
   - Group related variables
   - Separate environments
   - Isolate sensitive data

## 6. Common Pitfalls
> ❌ Issues to avoid in your code.

1. Hardcoding Values
```hcl
# Bad: Hardcoded values
instance_type = "t2.micro"

# Good: Use variables
instance_type = var.instance_type
```

2. Missing Defaults
```hcl
# Bad: No default or validation
variable "region" {
  type = string
}

# Good: Either default or validation
variable "region" {
  type    = string
  default = "us-west-2"
}
```

3. Wrong Type Usage
```hcl
# Bad: Mixed types
variable "ports" {
  type = list(any)
}

# Good: Specific type
variable "ports" {
  type = list(number)
}
```
> 🎯 Remember: Type specificity prevents errors.
```hcl
# String
variable "environment" {
  type    = string
  default = "development"
}

# Number
variable "instance_count" {
  type    = number
  default = 2
}

# Boolean
variable "enable_monitoring" {
  type    = bool
  default = true
}

# List
variable "availability_zones" {
  type    = list(string)
  default = ["us-west-2a", "us-west-2b"]
}

# Map
variable "instance_types" {
  type = map(string)
  default = {
    dev  = "t2.micro"
    prod = "t2.large"
  }
}
```

### Complex Types
```hcl
# Object
variable "vpc_settings" {
  type = object({
    cidr_block     = string
    dns_support    = bool
    instance_count = number
  })
}

# Tuple
variable "port_config" {
  type    = tuple([string, number, bool])
  default = ["tcp", 80, true]
}

# Set
variable "unique_tags" {
  type    = set(string)
  default = ["app", "web", "public"]
}
```

## 2. Variable Precedence (Highest to Lowest)
1. Command line flags (-var and -var-file)
2. *.auto.tfvars files
3. terraform.tfvars
4. Environment variables (TF_VAR_*)
5. Default values in variable declarations

## 3. Important Interview Questions & Answers

### Q1: What are the different ways to pass variables in Terraform?
A: Variables can be passed through:
1. Command line flags:
```bash
terraform apply -var="instance_type=t2.micro"
```

2. Variable definition files:
```hcl
# terraform.tfvars
instance_type = "t2.micro"
```

3. Environment variables:
```bash
export TF_VAR_instance_type="t2.micro"
```

4. Default values in variable declarations
```hcl
variable "instance_type" {
  default = "t2.micro"
}
```

### Q2: How do you handle sensitive variables?
A: Sensitive variables can be handled using:
```hcl
# Declaration
variable "db_password" {
  type      = string
  sensitive = true
}

# Usage
- Store in environment variables: export TF_VAR_db_password="secret"
- Use secrets management tools (AWS Secrets Manager, HashiCorp Vault)
- Never store sensitive values in version control
```

### Q3: Explain variable validation in Terraform
A: Variable validation helps ensure input values meet specific criteria:
```hcl
variable "environment" {
  type        = string
  description = "Environment name"
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

### Q4: How do you handle different configurations for multiple environments?
A: Use separate variable files:
```plaintext
project/
├── main.tf
├── variables.tf
├── dev.tfvars
└── prod.tfvars

# Apply with:
terraform apply -var-file="dev.tfvars"
```

### Q5: What are locals in Terraform and when should you use them?
A: Locals are named values used to avoid repetition:
```hcl
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    Terraform   = "true"
  }
  
  name_prefix = "${var.environment}-${var.project_name}"
}

resource "aws_instance" "example" {
  tags = local.common_tags
  name = "${local.name_prefix}-instance"
}
```

## 4. Advanced Concepts

### 1. Dynamic Blocks with Variables
```hcl
variable "ingress_rules" {
  type = list(object({
    port        = number
    protocol    = string
    cidr_blocks = list(string)
  }))
}

resource "aws_security_group" "example" {
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

### 2. Conditional Variable Usage
```hcl
variable "environment" {
  type = string
}

locals {
  instance_count = var.environment == "prod" ? 3 : 1
  instance_type  = var.environment == "prod" ? "t2.large" : "t2.micro"
}
```

### 3. Variable Type Constraints
```hcl
variable "instance_settings" {
  type = object({
    type  = string
    count = number
    tags  = map(string)
  })

  validation {
    condition     = can(regex("^t[23].", var.instance_settings.type))
    error_message = "Instance type must be t2 or t3 series."
  }
}
```

## 5. Best Practices

1. Variable Naming
   - Use snake_case for variable names
   - Be descriptive and consistent
   - Follow team conventions

2. Documentation
   - Always include descriptions
   - Document validation rules
   - Include examples in comments

3. Organization
   - Group related variables together
   - Use separate files for different environments
   - Keep sensitive variables separate

4. Validation
   - Implement validation rules for critical variables
   - Use custom error messages
   - Consider business logic in validations

## 6. Common Pitfalls to Avoid

1. Hardcoding Values
```hcl
# Bad
resource "aws_instance" "example" {
  instance_type = "t2.micro"  # Hardcoded
}

# Good
resource "aws_instance" "example" {
  instance_type = var.instance_type
}
```

2. Not Handling Defaults Properly
```hcl
# Bad - Missing default could cause errors
variable "region" {
  type = string
}

# Good - Either provide default or document that it's required
variable "region" {
  type        = string
  default     = "us-west-2"
  description = "AWS region for resources"
}
```

3. Incorrect Type Definitions
```hcl
# Bad - Mixed types in list
variable "ports" {
  type    = list(any)  # Avoid using 'any'
  default = [80, "443"]
}

# Good - Specific type
variable "ports" {
  type    = list(number)
  default = [80, 443]
}
```
