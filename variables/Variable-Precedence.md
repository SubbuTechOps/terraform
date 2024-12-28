# Terraform Variable Precedence Guide

## Table of Contents
1. [Understanding Variable Precedence](#1-understanding-variable-precedence)
2. [Practical Examples](#2-practical-examples)
3. [Real-World Scenarios](#3-real-world-scenarios)
4. [Common Issues and Solutions](#4-common-issues-and-solutions)
5. [Interview Questions and Answers](#5-interview-questions-and-answers)

## 1. Understanding Variable Precedence

Variable precedence in Terraform (highest to lowest):
1. Command line flags (-var and -var-file)
2. *.auto.tfvars files (alphabetically)
3. terraform.tfvars
4. Environment variables (TF_VAR_*)
5. Default values in variable declarations

## 2. Practical Examples

### Example 1: Multiple Sources of Variables

```hcl
# variables.tf
variable "environment" {
  type    = string
  default = "development"  # Precedence Level 5 (lowest)
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}

# terraform.tfvars (Precedence Level 3)
environment = "staging"
instance_type = "t2.medium"

# prod.auto.tfvars (Precedence Level 2)
environment = "production"
instance_type = "t2.large"

# Environment Variables (Precedence Level 4)
# export TF_VAR_environment="test"
# export TF_VAR_instance_type="t3.small"

# Command Line (Precedence Level 1 - highest)
# terraform apply -var="environment=prod" -var="instance_type=t3.large"
```

### Example 2: Multiple .tfvars Files

```hcl
# base.tfvars
region = "us-west-2"
instance_type = "t2.micro"

# dev.auto.tfvars
environment = "dev"
instance_type = "t2.small"

# prod.auto.tfvars
environment = "prod"
instance_type = "t2.large"

# Override with command line
# terraform apply -var-file="base.tfvars" -var="instance_type=t3.xlarge"
```

## 3. Real-World Scenarios

### Scenario 1: Multi-Environment Deployment
```hcl
# common.tfvars
project = "myapp"
team = "devops"

# dev.auto.tfvars
environment = "dev"
instance_type = "t2.micro"
min_capacity = 1
max_capacity = 3

# prod.auto.tfvars
environment = "prod"
instance_type = "t2.large"
min_capacity = 3
max_capacity = 10

# secrets.tfvars (not in version control)
db_password = "secure123"
api_key = "abcd1234"

# Usage:
# terraform apply -var-file="secrets.tfvars"
```

### Scenario 2: Override for Testing
```hcl
# Override production settings temporarily for testing
# terraform apply \
#   -var-file="prod.auto.tfvars" \
#   -var="instance_type=t2.micro" \
#   -var="min_capacity=1"
```

## 4. Common Issues and Solutions

### Issue 1: Conflicting Values
Problem: Variables set in multiple places causing unexpected values
```hcl
# terraform.tfvars
environment = "staging"

# dev.auto.tfvars
environment = "dev"

# Solution: Use terraform console to check final value
# $ terraform console
# > var.environment
```

### Issue 2: Sensitive Variables
Problem: Sensitive values in version control
```hcl
# Bad Practice
# terraform.tfvars
db_password = "secretpass"  # Don't do this!

# Good Practice
# Use environment variables
# export TF_VAR_db_password="secretpass"
```

## 5. Interview Questions and Answers

### Basic Level Questions

Q1: "What is variable precedence in Terraform?"
> A: Variable precedence in Terraform defines the order in which variable values are resolved when defined in multiple places. The order from highest to lowest precedence is:
> 1. Command-line flags (-var and -var-file)
> 2. *.auto.tfvars files
> 3. terraform.tfvars
> 4. Environment variables (TF_VAR_*)
> 5. Default values in variable declarations

Q2: "Why would you use different .tfvars files?"
> A: Different .tfvars files are used to:
> - Separate environment-specific configurations (dev.tfvars, prod.tfvars)
> - Keep sensitive values separate (secrets.tfvars)
> - Share common configurations (common.tfvars)
> - Maintain cleaner and more organized code

### Intermediate Level Questions

Q3: "How would you handle multiple .auto.tfvars files and explain their precedence?"
> A: Multiple .auto.tfvars files are loaded alphabetically. For example:
```hcl
# 1_base.auto.tfvars
environment = "dev"

# 2_override.auto.tfvars
environment = "prod"

# The value from 2_override.auto.tfvars takes precedence due to alphabetical order
```

Q4: "In a CI/CD pipeline, how would you manage variable precedence across different environments?"
> A: In a CI/CD pipeline, you would typically:
> 1. Use base variables in terraform.tfvars
> 2. Override with environment-specific *.auto.tfvars
> 3. Use pipeline variables for sensitive values
> 4. Example:
```yaml
# CI/CD Pipeline
stages:
  - name: 'Deploy to Production'
    steps:
      - script: |
          export TF_VAR_db_password=$(DB_PASSWORD)
          terraform apply -var-file="prod.auto.tfvars"
```

### Advanced Level Questions

Q5: "How would you debug variable precedence issues in a complex setup?"
> A: To debug variable precedence:
> 1. Use terraform console to check final values
> 2. Use -var-file precedence
> 3. Example debugging process:
```bash
# Check final value
terraform console
> var.environment

# Check which files are being loaded
terraform apply -var-file="base.tfvars" -var-file="override.tfvars" -debug

# Use tfvars files explicitly
terraform apply \
  -var-file="common.tfvars" \
  -var-file="env-specific.tfvars" \
  -var-file="override.tfvars"
```

Q6: "How would you handle variable precedence in a multi-workspace environment with shared variables?"
> A: For multi-workspace environments:
```hcl
# common.tfvars
project = "myapp"

# workspace variables
variable "workspace_config" {
  type = map(object({
    environment = string
    region     = string
  }))
  default = {
    dev = {
      environment = "development"
      region     = "us-west-2"
    }
    prod = {
      environment = "production"
      region     = "us-east-1"
    }
  }
}

# Usage
locals {
  config = var.workspace_config[terraform.workspace]
}
```

### Expert Level Questions

Q7: "Design a system for managing variable precedence across multiple teams and environments while ensuring security and compliance."
> A: Solution approach:
```hcl
# 1. Base structure
variable "team_configs" {
  type = map(object({
    allowed_environments = list(string)
    resource_prefix     = string
    compliance_tags     = map(string)
  }))
}

# 2. Environment overrides
variable "env_configs" {
  type = map(object({
    allowed_teams     = list(string)
    compliance_level  = string
    required_tags     = list(string)
  }))
}

# 3. Validation
variable "deployment_config" {
  type = object({
    team = string
    environment = string
  })
  
  validation {
    condition = contains(
      var.team_configs[var.deployment_config.team].allowed_environments,
      var.deployment_config.environment
    )
    error_message = "Team not authorized for this environment."
  }
}

# 4. Implementation
locals {
  effective_config = {
    team_settings = var.team_configs[var.deployment_config.team]
    env_settings  = var.env_configs[var.deployment_config.environment]
    required_tags = merge(
      local.team_settings.compliance_tags,
      {
        Environment = var.deployment_config.environment
        Team        = var.deployment_config.team
      }
    )
  }
}
```

Q8: "How would you implement a custom variable precedence system for special use cases?"
> A: Implementation example:
```hcl
locals {
  # Custom precedence logic
  custom_precedence = {
    environment = coalesce(
      var.override_environment,
      lookup(local.workspace_mappings, terraform.workspace, null),
      var.default_environment
    )
    instance_type = coalesce(
      var.override_instance_type,
      lookup(local.env_instance_types, local.environment, null),
      var.default_instance_type
    )
  }
  
  # Helper functions
  workspace_mappings = {
    dev  = "development"
    stg  = "staging"
    prod = "production"
  }
  
  env_instance_types = {
    development = "t2.micro"
    staging     = "t2.medium"
    production  = "t2.large"
  }
}
```

These interview questions and answers cover the range from basic understanding to complex implementations of variable precedence in Terraform.
