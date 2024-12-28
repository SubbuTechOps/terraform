# Terraform Variables - Basic Concepts and Examples

## Basic Variable Types and Declaration

### 1. String Variables
```hcl
# variables.tf
variable "environment" {
  type        = string
  description = "Environment name"
  default     = "development"
}

# Usage
resource "aws_instance" "example" {
  tags = {
    Environment = var.environment
  }
}
```

### 2. Number Variables
```hcl
variable "instance_count" {
  type        = number
  description = "Number of EC2 instances"
  default     = 2
}

# Usage
resource "aws_instance" "cluster" {
  count = var.instance_count
  # ... other configurations
}
```

### 3. Boolean Variables
```hcl
variable "enable_monitoring" {
  type        = bool
  description = "Enable CloudWatch monitoring"
  default     = true
}

# Usage
resource "aws_instance" "example" {
  monitoring = var.enable_monitoring
}
```

### 4. List Variables
```hcl
variable "subnet_cidr_blocks" {
  type        = list(string)
  description = "List of subnet CIDR blocks"
  default     = ["10.0.1.0/24", "10.0.2.0/24"]
}

# Usage
resource "aws_subnet" "example" {
  count             = length(var.subnet_cidr_blocks)
  cidr_block        = var.subnet_cidr_blocks[count.index]
  vpc_id            = aws_vpc.main.id
}
```

### 5. Map Variables
```hcl
variable "instance_types" {
  type        = map(string)
  description = "Instance types per environment"
  default     = {
    dev  = "t2.micro"
    prod = "t2.large"
  }
}

# Usage
resource "aws_instance" "example" {
  instance_type = var.instance_types[var.environment]
}
```

### 6. Object Variables
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

# Usage
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_settings.cidr_block
  enable_dns_support   = var.vpc_settings.dns_support
}
```

## Variable Definition Files

### terraform.tfvars
```hcl
# terraform.tfvars
environment = "production"
instance_count = 4
enable_monitoring = true
```

### Environment-specific files
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

## Variable Validation

### Basic Validation
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

### Complex Validation
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

## Ways to Set Variables

### 1. Command Line
```bash
terraform apply -var="environment=prod" -var="instance_count=5"
```

### 2. Variable Files
```bash
terraform apply -var-file="prod.tfvars"
```

### 3. Environment Variables
```bash
export TF_VAR_environment="prod"
export TF_VAR_instance_count="5"
terraform apply
```

## Variable Precedence (Highest to Lowest)
1. Command line flags (-var and -var-file)
2. *.auto.tfvars files
3. terraform.tfvars
4. Environment variables (TF_VAR_*)
5. Default values in variable declarations

## Local Values (locals)
```hcl
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    Terraform   = "true"
  }
  
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

## Working with Sensitive Variables
```hcl
variable "database_password" {
  type        = string
  sensitive   = true
  description = "Database password"
}

# Store in terraform.tfvars (don't commit to version control)
database_password = "mysecretpassword"

# Or use environment variables
# export TF_VAR_database_password="mysecretpassword"
```
