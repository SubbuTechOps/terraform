# Terraform Variables - Basic Concepts with Notes

## Table of Contents
1. [Basic Variable Types and Declaration](#1-basic-variable-types-and-declaration)
   - [String Variables](#11-string-variables)
   - [Number Variables](#12-number-variables)
   - [Boolean Variables](#13-boolean-variables)
   - [List Variables](#14-list-variables)
   - [Map Variables](#15-map-variables)
   - [Object Variables](#16-object-variables)
2. [Variable Definition Files](#2-variable-definition-files)
   - [terraform.tfvars](#21-terraformtfvars)
   - [Environment-specific files](#22-environment-specific-files)
3. [Variable Validation](#3-variable-validation)
   - [Basic Validation](#31-basic-validation)
   - [Complex Validation](#32-complex-validation)
4. [Ways to Set Variables](#4-ways-to-set-variables)
   - [Command Line](#41-command-line)
   - [Variable Files](#42-variable-files)
   - [Environment Variables](#43-environment-variables)
5. [Variable Precedence](#5-variable-precedence-highest-to-lowest)
6. [Local Values](#6-local-values)
7. [Working with Sensitive Variables](#7-working-with-sensitive-variables)
8. [Best Practices Summary](#8-best-practices-summary)

## 1. Basic Variable Types and Declaration

### 1.1 String Variables
> Note: String variables are used for text values like names, descriptions, or any textual configuration.
```hcl
variable "environment" {
  type        = string
  description = "Environment name"
  default     = "development"
}

# Usage example - Tags an AWS resource with the environment name
resource "aws_instance" "example" {
  tags = {
    Environment = var.environment
  }
}
```

### 1.2 Number Variables
> Note: Number variables are used for quantities, sizes, or any numerical configuration like instance counts or port numbers.
```hcl
variable "instance_count" {
  type        = number
  description = "Number of EC2 instances"
  default     = 2
}

# Usage example - Creates multiple instances based on count
resource "aws_instance" "cluster" {
  count = var.instance_count
  # ... other configurations
}
```

### 1.3 Boolean Variables
> Note: Boolean variables are used for yes/no or true/false configurations, perfect for enabling/disabling features.
```hcl
variable "enable_monitoring" {
  type        = bool
  description = "Enable CloudWatch monitoring"
  default     = true
}

# Usage example - Enables/disables monitoring based on boolean value
resource "aws_instance" "example" {
  monitoring = var.enable_monitoring
}
```

### 1.4 List Variables
> Note: List variables store multiple values of the same type in an ordered list, great for multiple similar items like subnet CIDR blocks.
```hcl
variable "subnet_cidr_blocks" {
  type        = list(string)
  description = "List of subnet CIDR blocks"
  default     = ["10.0.1.0/24", "10.0.2.0/24"]
}

# Usage example - Creates multiple subnets from the list
resource "aws_subnet" "example" {
  count             = length(var.subnet_cidr_blocks)
  cidr_block        = var.subnet_cidr_blocks[count.index]
  vpc_id            = aws_vpc.main.id
}
```

### 1.5 Map Variables
> Note: Map variables store key-value pairs, perfect for lookup tables or environment-specific configurations.
```hcl
variable "instance_types" {
  type        = map(string)
  description = "Instance types per environment"
  default     = {
    dev  = "t2.micro"
    prod = "t2.large"
  }
}

# Usage example - Selects instance type based on environment
resource "aws_instance" "example" {
  instance_type = var.instance_types[var.environment]
}
```

### 1.6 Object Variables
> Note: Object variables are complex types combining multiple values of different types, useful for grouped configurations.
```hcl
variable "vpc_settings" {
  type = object({
    cidr_block = string
    dns_support = bool
    subnet_count = number
  })
  description = "VPC configuration settings"
  default = {
    cidr_block = "10.0.0.0/16"
    dns_support = true
    subnet_count = 2
  }
}

# Usage example - Configures VPC using object properties
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_settings.cidr_block
  enable_dns_support   = var.vpc_settings.dns_support
}
```

## 2. Variable Definition Files
> Note: Variable files help organize and separate variable values for different environments or purposes.

### 2.1 terraform.tfvars
> Note: The default file for setting variable values, automatically loaded by Terraform.
```hcl
# terraform.tfvars
environment = "production"
instance_count = 4
enable_monitoring = true
```

### 2.2 Environment-specific files
> Note: Separate files for different environments, must be explicitly loaded with -var-file flag.
```hcl
# prod.tfvars
environment = "production"
instance_count = 4
enable_monitoring = true

# dev.tfvars
environment = "development"
instance_count = 1
enable_monitoring = false
```

## 3. Variable Validation
> Note: Validation helps ensure variable values meet specific requirements or constraints.

### 3.1 Basic Validation
> Note: Simple validation for checking allowed values.
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

### 3.2 Complex Validation
> Note: More sophisticated validation combining multiple conditions.
```hcl
variable "instance_settings" {
  type = object({
    instance_type = string
    volume_size   = number
  })
  
  validation {
    condition     = can(regex("^t[23].", var.instance_settings.instance_type))
    error_message = "Instance type must be t2 or t3 series."
  }
  
  validation {
    condition     = var.instance_settings.volume_size >= 20
    error_message = "Volume size must be at least 20 GB."
  }
}
```

## 4. Ways to Set Variables
> Note: Multiple methods to set variable values, following a specific precedence order.

### 4.1 Command Line
> Note: Highest precedence, good for one-time overrides.
```bash
terraform apply -var="environment=prod" -var="instance_count=5"
```

### 4.2 Variable Files
> Note: Good for environment-specific configurations.
```bash
terraform apply -var-file="prod.tfvars"
```

### 4.3 Environment Variables
> Note: Good for sensitive values and CI/CD pipelines.
```bash
export TF_VAR_environment="prod"
export TF_VAR_instance_count="5"
terraform apply
```

## 5. Variable Precedence (Highest to Lowest)
> Note: Understanding precedence is crucial when variables are set in multiple places.
1. Command line flags (-var and -var-file)
2. *.auto.tfvars files
3. terraform.tfvars
4. Environment variables (TF_VAR_*)
5. Default values in variable declarations

## 6. Local Values (locals)
> Note: Locals help compute derived values and reduce repetition in your configuration.
```hcl
locals {
  # Common tags for all resources
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    Terraform   = "true"
  }
  
  # Computed value based on multiple variables
  instance_name = "${var.environment}-${var.project_name}-instance"
}

resource "aws_instance" "example" {
  tags = merge(
    local.common_tags,
    {
      Name = local.instance_name
    }
  )
}
```

## 7. Working with Sensitive Variables
> Note: Special handling for sensitive values like passwords or API keys.
```hcl
variable "database_password" {
  type        = string
  sensitive   = true  # Masks the value in logs and output
  description = "Database password"
}

# Store in terraform.tfvars (don't commit to version control)
database_password = "mysecretpassword"

# Or better, use environment variables
# export TF_VAR_database_password="mysecretpassword"
```

## 8. Best Practices Summary
> Note: Key guidelines for working with Terraform variables.

1. Always include descriptions for variables
2. Use validation where possible to catch errors early
3. Keep sensitive values out of version control
4. Use consistent naming conventions
5. Group related variables together
6. Use appropriate variable types for different data
7. Leverage locals for computed values
8. Document variable requirements in README
