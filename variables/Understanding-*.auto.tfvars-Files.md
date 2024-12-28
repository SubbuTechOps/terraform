# Understanding *.auto.tfvars Files in Terraform

## 1. What are *.auto.tfvars Files?
> 💡 Note: Files with names ending in `.auto.tfvars` or `.auto.tfvars.json` are automatically loaded by Terraform.

## 2. Project Structure
```plaintext
project/
├── main.tf
├── variables.tf
├── terraform.tfvars        # Common variables
├── dev.auto.tfvars        # Development environment
├── staging.auto.tfvars    # Staging environment
└── prod.auto.tfvars       # Production environment
```

## 3. Example Configuration

### Basic Variable Declarations (variables.tf)
```hcl
# variables.tf
variable "environment" {
  type        = string
  description = "Environment name"
}

variable "instance_config" {
  type = object({
    type  = string
    count = number
  })
  description = "EC2 instance configuration"
}

variable "vpc_cidr" {
  type        = string
  description = "VPC CIDR block"
}

variable "enable_monitoring" {
  type        = bool
  description = "Enable detailed monitoring"
}
```

### Common Variables (terraform.tfvars)
```hcl
# terraform.tfvars - Common configurations for all environments
project_name = "my-webapp"
region       = "us-west-2"
common_tags = {
  Project     = "MyWebApp"
  Terraform   = "true"
  Department  = "DevOps"
}
```

### Environment-Specific Files

#### Development Environment (dev.auto.tfvars)
```hcl
# dev.auto.tfvars
environment = "development"
instance_config = {
  type  = "t2.micro"
  count = 1
}
vpc_cidr          = "10.0.0.0/16"
enable_monitoring = false

additional_tags = {
  Environment = "dev"
  CostCenter  = "dev-12345"
}
```

#### Staging Environment (staging.auto.tfvars)
```hcl
# staging.auto.tfvars
environment = "staging"
instance_config = {
  type  = "t2.medium"
  count = 2
}
vpc_cidr          = "172.16.0.0/16"
enable_monitoring = true

additional_tags = {
  Environment = "staging"
  CostCenter  = "stg-12345"
}
```

#### Production Environment (prod.auto.tfvars)
```hcl
# prod.auto.tfvars
environment = "production"
instance_config = {
  type  = "t2.large"
  count = 3
}
vpc_cidr          = "192.168.0.0/16"
enable_monitoring = true

additional_tags = {
  Environment = "prod"
  CostCenter  = "prod-12345"
}
```

## 4. Using the Configuration (main.tf)
```hcl
# main.tf
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  
  tags = merge(
    var.common_tags,
    var.additional_tags,
    {
      Name = "${var.project_name}-${var.environment}-vpc"
    }
  )
}

resource "aws_instance" "app" {
  count = var.instance_config.count
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_config.type
  monitoring    = var.enable_monitoring
  
  tags = merge(
    var.common_tags,
    var.additional_tags,
    {
      Name = "${var.project_name}-${var.environment}-instance-${count.index + 1}"
    }
  )
}
```

## 5. Applying the Configuration

### For Development Environment
```bash
# Terraform automatically loads dev.auto.tfvars
terraform apply
```

### For Production Environment
```bash
# Terraform automatically loads prod.auto.tfvars
terraform apply
```

## 6. Key Points to Remember

1. Loading Order
   > 🔑 Note: .auto.tfvars files are loaded alphabetically

2. Common Use Cases
   - Environment-specific configurations
   - Regional settings
   - Resource sizing
   - Feature flags

3. Best Practices
   ```hcl
   # Use clear naming conventions
   dev.auto.tfvars
   staging.auto.tfvars
   prod.auto.tfvars
   
   # Use consistent structure across files
   # Bad
   dev.auto.tfvars:
   app_size = "small"
   
   prod.auto.tfvars:
   instance_size = "large"  # Different naming
   
   # Good
   dev.auto.tfvars:
   instance_size = "small"
   
   prod.auto.tfvars:
   instance_size = "large"  # Consistent naming
   ```

4. Variable Validation
```hcl
# variables.tf
variable "environment" {
  type = string
  validation {
    condition     = contains(["development", "staging", "production"], var.environment)
    error_message = "Environment must be development, staging, or production."
  }
}
```

## 7. Working with Multiple Environments

### Using Workspaces
```bash
# Create workspace
terraform workspace new dev
terraform workspace new prod

# Select workspace
terraform workspace select dev
terraform apply  # Uses dev.auto.tfvars

terraform workspace select prod
terraform apply  # Uses prod.auto.tfvars
```

### Using Different Backends
```hcl
# backend.tf
backend "s3" {
  bucket = "terraform-state"
  key    = "${var.environment}/terraform.tfstate"
  region = "us-west-2"
}
```

## 8. Common Patterns and Examples

### Feature Toggles
```hcl
# dev.auto.tfvars
features = {
  monitoring   = false
  backup      = false
  encryption  = false
}

# prod.auto.tfvars
features = {
  monitoring   = true
  backup      = true
  encryption  = true
}
```

### Resource Scaling
```hcl
# dev.auto.tfvars
scaling = {
  min     = 1
  max     = 2
  desired = 1
}

# prod.auto.tfvars
scaling = {
  min     = 3
  max     = 10
  desired = 5
}
```
