# Handling Environment-Specific Variables in Terraform

## Table of Contents
1. [Project Structure](#1-project-structure)
2. [Variable Declarations](#2-variable-declarations)
3. [Environment-Specific Files](#3-environment-specific-files)
4. [Command Usage](#4-command-usage)
5. [Best Practices](#5-best-practices)
6. [Alternative Approaches](#6-alternative-approaches)
7. [Common Scenarios](#7-common-scenarios)

## 1. Project Structure
```plaintext
project/
├── main.tf
├── variables.tf
├── outputs.tf
├── common.tfvars      # Common variables for all environments
├── dev.tfvars         # Development environment variables
├── staging.tfvars     # Staging environment variables
└── prod.tfvars        # Production environment variables
```

## 2. Variable Declarations
```hcl
# variables.tf
variable "environment" {
  type        = string
  description = "Environment name (dev/staging/prod)"
}

variable "instance_type" {
  type        = string
  description = "EC2 instance type"
}

variable "vpc_cidr" {
  type        = string
  description = "VPC CIDR block"
}

variable "instance_count" {
  type        = number
  description = "Number of instances to create"
}

variable "enable_monitoring" {
  type        = bool
  description = "Enable detailed monitoring"
}
```

## 3. Environment-Specific Files

### Common Variables
```hcl
# common.tfvars
project_name    = "my-webapp"
company_name    = "acme-corp"
region          = "us-west-2"
```

### Development Environment
```hcl
# dev.tfvars
environment       = "development"
instance_type     = "t2.micro"
vpc_cidr          = "10.0.0.0/16"
instance_count    = 1
enable_monitoring = false
```

### Production Environment
```hcl
# prod.tfvars
environment       = "production"
instance_type     = "t2.large"
vpc_cidr          = "172.16.0.0/16"
instance_count    = 3
enable_monitoring = true
```

## 4. Command Usage

### Basic Command Structure
```bash
# Planning with specific environment
terraform plan -var-file="common.tfvars" -var-file="dev.tfvars"

# Applying with specific environment
terraform apply -var-file="common.tfvars" -var-file="prod.tfvars"
```

### Using Multiple Variable Files
```bash
terraform apply \
  -var-file="common.tfvars" \
  -var-file="dev.tfvars" \
  -var="instance_count=2"
```

## 5. Best Practices

1. File Organization
   - Keep common variables in `common.tfvars`
   - Use environment-specific files for environment-specific values
   - Maintain consistent file naming

2. Variable Precedence
   - Remember that later var-files override earlier ones
   - Command-line vars override var-files

3. Documentation
   ```markdown
   # README.md
   ## Usage
   For development:
   ```bash
   terraform apply -var-file="common.tfvars" -var-file="dev.tfvars"
   ```
   For production:
   ```bash
   terraform apply -var-file="common.tfvars" -var-file="prod.tfvars"
   ```
   ```

## 6. Alternative Approaches

### 1. Using Shell Scripts
```bash
#!/bin/bash
# deploy.sh

environment=$1
if [ -z "$environment" ]; then
    echo "Usage: ./deploy.sh <environment>"
    echo "Environments: dev, staging, prod"
    exit 1
fi

if [ ! -f "${environment}.tfvars" ]; then
    echo "Error: ${environment}.tfvars not found"
    exit 1
fi

terraform apply \
    -var-file="common.tfvars" \
    -var-file="${environment}.tfvars"
```

### 2. Using Workspaces with tfvars
```bash
#!/bin/bash
# deploy-workspace.sh

environment=$1
if [ -z "$environment" ]; then
    echo "Usage: ./deploy-workspace.sh <environment>"
    exit 1
fi

terraform workspace select $environment || terraform workspace new $environment
terraform apply -var-file="common.tfvars" -var-file="${environment}.tfvars"
```

## 7. Common Scenarios

### Scenario 1: Different Resource Configurations
```hcl
# dev.tfvars
environment = "development"
resources = {
  web = {
    instance_type = "t2.micro"
    count        = 1
  }
  db = {
    instance_type = "t2.small"
    storage_size = 20
  }
}

# prod.tfvars
environment = "production"
resources = {
  web = {
    instance_type = "t2.large"
    count        = 3
  }
  db = {
    instance_type = "t2.xlarge"
    storage_size = 100
  }
}
```

### Scenario 2: Regional Configurations
```hcl
# dev.tfvars
environment = "development"
regions = {
  primary = {
    region = "us-west-2"
    azs    = ["us-west-2a", "us-west-2b"]
  }
}

# prod.tfvars
environment = "production"
regions = {
  primary = {
    region = "us-west-2"
    azs    = ["us-west-2a", "us-west-2b", "us-west-2c"]
  }
  dr = {
    region = "us-east-1"
    azs    = ["us-east-1a", "us-east-1b"]
  }
}
```

### Scenario 3: Tags and Naming
```hcl
# dev.tfvars
environment = "development"
tags = {
  Environment = "development"
  CostCenter  = "dev-team"
  Terraform   = "true"
}
name_prefix = "dev"

# prod.tfvars
environment = "production"
tags = {
  Environment = "production"
  CostCenter  = "prod-team"
  Terraform   = "true"
  Backup      = "true"
}
name_prefix = "prod"
```
