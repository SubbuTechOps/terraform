# Comprehensive Guide to Terraform Variables

## Table of Contents
1. Introduction
2. Types of Variables
3. Variable Declaration and Usage
4. Variable Precedence
5. Real-World Scenarios
6. Best Practices
7. Common Issues and Solutions

## 1. Introduction

Terraform variables are fundamental components that enable infrastructure code to be flexible, reusable, and maintainable. This guide covers everything you need to know about Terraform variables with practical examples and real-world scenarios.

## 2. Types of Variables

### 2.1 String Variables
```hcl
variable "environment" {
  type        = string
  description = "Environment name (dev/staging/prod)"
  default     = "dev"
}
```

### 2.2 Number Variables
```hcl
variable "instance_count" {
  type        = number
  description = "Number of instances to create"
  default     = 2
}
```

### 2.3 Boolean Variables
```hcl
variable "enable_monitoring" {
  type        = bool
  description = "Enable detailed monitoring"
  default     = false
}
```

### 2.4 List Variables
```hcl
variable "availability_zones" {
  type        = list(string)
  description = "List of availability zones"
  default     = ["us-west-2a", "us-west-2b"]
}
```

### 2.5 Map Variables
```hcl
variable "instance_types" {
  type        = map(string)
  description = "Instance types per environment"
  default     = {
    dev     = "t2.micro"
    staging = "t2.medium"
    prod    = "t2.large"
  }
}
```

### 2.6 Object Variables
```hcl
variable "vpc_config" {
  type = object({
    cidr_block = string
    subnets    = list(object({
      cidr_block = string
      zone       = string
      public     = bool
    }))
  })
  description = "VPC configuration"
}
```

## 3. Variable Declaration and Usage

### 3.1 Basic Variable Declaration
```hcl
# variables.tf
variable "project_name" {
  type        = string
  description = "Name of the project"
  default     = "my-terraform-project"
}

# Usage in main.tf
resource "aws_instance" "example" {
  tags = {
    Project = var.project_name
  }
}
```

### 3.2 Variable Validation
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

## 4. Variable Precedence (Highest to Lowest)

1. Command-line flags (-var and -var-file)
2. *.auto.tfvars files
3. terraform.tfvars
4. Environment variables (TF_VAR_*)
5. Default values in variable declarations

## 5. Real-World Scenarios

### Scenario 1: Multi-Environment AWS Infrastructure

```hcl
# variables.tf
variable "environment" {
  type        = string
  description = "Deployment environment"
}

variable "vpc_settings" {
  type = object({
    cidr_block = string
    subnets    = map(object({
      cidr_block = string
      public     = bool
    }))
  })
}

# environments/dev.tfvars
environment = "dev"
vpc_settings = {
  cidr_block = "10.0.0.0/16"
  subnets = {
    subnet1 = {
      cidr_block = "10.0.1.0/24"
      public     = true
    }
    subnet2 = {
      cidr_block = "10.0.2.0/24"
      public     = false
    }
  }
}

# main.tf
resource "aws_vpc" "main" {
  cidr_block = var.vpc_settings.cidr_block
  
  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

resource "aws_subnet" "subnets" {
  for_each = var.vpc_settings.subnets
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = each.value.cidr_block
  map_public_ip_on_launch = each.value.public
  
  tags = {
    Name        = "${var.environment}-${each.key}"
    Environment = var.environment
  }
}
```

### Scenario 2: Dynamic ECS Cluster Configuration

```hcl
# variables.tf
variable "ecs_config" {
  type = object({
    cluster_name = string
    services     = map(object({
      name           = string
      container_port = number
      cpu           = number
      memory        = number
      desired_count = number
    }))
  })
  description = "ECS cluster configuration"
}

# prod.auto.tfvars
ecs_config = {
  cluster_name = "production-cluster"
  services = {
    web = {
      name           = "web-service"
      container_port = 80
      cpu           = 256
      memory        = 512
      desired_count = 3
    }
    api = {
      name           = "api-service"
      container_port = 8080
      cpu           = 512
      memory        = 1024
      desired_count = 2
    }
  }
}

# main.tf
resource "aws_ecs_cluster" "main" {
  name = var.ecs_config.cluster_name
}

resource "aws_ecs_service" "services" {
  for_each = var.ecs_config.services
  
  name            = each.value.name
  cluster         = aws_ecs_cluster.main.id
  desired_count   = each.value.desired_count
  
  # ... other service configurations
}
```

### Scenario 3: Dynamic Security Group Rules

```hcl
variable "security_rules" {
  type = map(object({
    description = string
    port        = number
    cidr_blocks = list(string)
    protocol    = string
  }))
  
  default = {
    http = {
      description = "HTTP Access"
      port        = 80
      cidr_blocks = ["0.0.0.0/0"]
      protocol    = "tcp"
    }
    https = {
      description = "HTTPS Access"
      port        = 443
      cidr_blocks = ["0.0.0.0/0"]
      protocol    = "tcp"
    }
  }
}

resource "aws_security_group" "main" {
  name        = "dynamic-security-group"
  description = "Security group with dynamic rules"
  vpc_id      = aws_vpc.main.id
  
  dynamic "ingress" {
    for_each = var.security_rules
    content {
      description = ingress.value.description
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

## 6. Best Practices

1. **Variable Organization**
   - Keep variables organized in separate files based on their purpose
   - Use consistent naming conventions
   - Group related variables together

2. **Documentation**
   - Always include detailed descriptions for variables
   - Document any constraints or dependencies
   - Include examples in comments for complex variable structures

3. **Security**
   - Never commit sensitive values to version control
   - Use environment variables or secure vaults for secrets
   - Implement proper variable validation

4. **Maintainability**
   - Use default values where appropriate
   - Implement proper variable validation
   - Keep variable structures consistent across environments

5. **Version Control**
   - Include example .tfvars files in version control
   - Use .gitignore for environment-specific .tfvars files
   - Document variable requirements in README

## 7. Advanced Scenarios and Solutions for Experienced DevOps Engineers

### Scenario 1: Dynamic Resource Creation Based on Environment
Q: "We need to deploy different numbers of resources across environments with varying configurations. How would you structure your Terraform variables to handle this efficiently?"

A: Here's an implementation using dynamic configurations:

```hcl
variable "environment_configs" {
  type = map(object({
    instance_count = number
    instance_types = list(string)
    scaling_rules = map(object({
      metric_name    = string
      target_value   = number
      scale_in_cool  = number
      scale_out_cool = number
    }))
  }))

  default = {
    dev = {
      instance_count = 2
      instance_types = ["t3.micro"]
      scaling_rules = {
        cpu = {
          metric_name    = "CPUUtilization"
          target_value   = 70
          scale_in_cool  = 300
          scale_out_cool = 180
        }
      }
    }
    prod = {
      instance_count = 6
      instance_types = ["t3.large", "t3.xlarge"]
      scaling_rules = {
        cpu = {
          metric_name    = "CPUUtilization"
          target_value   = 60
          scale_in_cool  = 600
          scale_out_cool = 300
        }
        memory = {
          metric_name    = "MemoryUtilization"
          target_value   = 65
          scale_in_cool  = 600
          scale_out_cool = 300
        }
      }
    }
  }
}

# Usage
resource "aws_autoscaling_group" "main" {
  count = var.environment_configs[terraform.workspace].instance_count
  
  mixed_instances_policy {
    instances_distribution {
      on_demand_base_capacity = 1
      spot_allocation_strategy = "capacity-optimized"
    }
    
    launch_template {
      launch_template_specification {
        launch_template_id = aws_launch_template.main.id
        version           = "$Latest"
      }
      
      dynamic "override" {
        for_each = var.environment_configs[terraform.workspace].instance_types
        content {
          instance_type = override.value
        }
      }
    }
  }
}
```

### Scenario 2: Cross-Stack Variable Sharing
Q: "How would you handle variable sharing between multiple Terraform stacks while maintaining environment isolation?"

A: Here's an approach using output variables and data sources:

```hcl
# networking/outputs.tf
output "vpc_config" {
  value = {
    vpc_id = aws_vpc.main.id
    private_subnets = {
      for subnet in aws_subnet.private :
      subnet.availability_zone => {
        id                = subnet.id
        cidr_block       = subnet.cidr_block
        route_table_id   = aws_route_table.private[subnet.availability_zone].id
        nat_gateway_ip   = aws_nat_gateway.main[subnet.availability_zone].public_ip
      }
    }
    security_groups = {
      for sg in aws_security_group.main :
      sg.name => {
        id    = sg.id
        rules = sg.ingress[*].from_port
      }
    }
  }
}

# app/variables.tf
variable "stack_configs" {
  type = object({
    networking = object({
      workspace = string
      key       = string
    })
  })
}

# app/data.tf
data "terraform_remote_state" "networking" {
  backend = "s3"
  config = {
    bucket  = "terraform-state"
    key     = var.stack_configs.networking.key
    workspace = var.stack_configs.networking.workspace
  }
}

locals {
  vpc_config = data.terraform_remote_state.networking.outputs.vpc_config
}
```

### Scenario 3: Dynamic IAM Policy Generation
Q: "How would you structure variables to generate IAM policies dynamically based on service requirements?"

A: Here's a solution using dynamic policy generation:

```hcl
variable "service_iam_configs" {
  type = map(object({
    service_name = string
    permissions = map(object({
      actions     = list(string)
      resources   = list(string)
      conditions = optional(list(object({
        test     = string
        variable = string
        values   = list(string)
      })))
    }))
  }))

  default = {
    "api-service" = {
      service_name = "API Service"
      permissions = {
        s3 = {
          actions   = ["s3:GetObject", "s3:PutObject"]
          resources = ["arn:aws:s3:::app-bucket/*"]
          conditions = [{
            test     = "StringEquals"
            variable = "aws:RequestTag/Environment"
            values   = ["prod"]
          }]
        }
        sqs = {
          actions   = ["sqs:SendMessage"]
          resources = ["arn:aws:sqs:*:*:app-queue"]
        }
      }
    }
  }
}

# Usage
data "aws_iam_policy_document" "service_policies" {
  for_each = var.service_iam_configs

  dynamic "statement" {
    for_each = each.value.permissions
    content {
      actions   = statement.value.actions
      resources = statement.value.resources
      
      dynamic "condition" {
        for_each = statement.value.conditions != null ? statement.value.conditions : []
        content {
          test     = condition.value.test
          variable = condition.value.variable
          values   = condition.value.values
        }
      }
    }
  }
}
```

### Scenario 4: Complex Validation Logic
Q: "How would you implement complex validation rules for infrastructure configurations?"

A: Here's an example with sophisticated validation:

```hcl
variable "cluster_config" {
  type = object({
    min_size         = number
    max_size         = number
    instance_types   = list(string)
    node_labels      = map(string)
    taints          = list(object({
      key    = string
      value  = string
      effect = string
    }))
  })

  validation {
    condition = (
      var.cluster_config.min_size <= var.cluster_config.max_size &&
      var.cluster_config.min_size > 0
    )
    error_message = "Min size must be positive and less than or equal to max size."
  }

  validation {
    condition = alltrue([
      for type in var.cluster_config.instance_types :
      can(regex("^[t2|t3|m4|m5|c4|c5].*", type))
    ])
    error_message = "Only t2, t3, m4, m5, c4, and c5 instance families are allowed."
  }

  validation {
    condition = length(setintersection(
      keys(var.cluster_config.node_labels),
      ["kubernetes.io", "k8s.io", "eks.amazonaws.com"]
    )) == 0
    error_message = "Reserved label prefixes are not allowed in node labels."
  }
}
```

### Scenario 5: Advanced Resource Scheduling
Q: "How would you implement complex resource scheduling using Terraform variables?"

A: Here's a solution for scheduling resources:

```hcl
variable "resource_schedule" {
  type = map(object({
    start_time = string
    stop_time  = string
    days       = list(string)
    scaling = object({
      min_size = number
      max_size = number
      desired  = number
    })
    exceptions = list(object({
      date        = string
      description = string
      scaling = object({
        min_size = number
        max_size = number
        desired  = number
      })
    }))
  }))

  default = {
    business_hours = {
      start_time = "08:00"
      stop_time  = "18:00"
      days       = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"]
      scaling = {
        min_size = 2
        max_size = 10
        desired  = 4
      }
      exceptions = [
        {
          date        = "2024-12-25"
          description = "Christmas Day"
          scaling = {
            min_size = 1
            max_size = 2
            desired  = 1
          }
        }
      ]
    }
  }
}

# Usage with AWS EventBridge
resource "aws_cloudwatch_event_rule" "schedule" {
  for_each = var.resource_schedule
  
  name                = "schedule-${each.key}"
  description         = "Schedule for ${each.key}"
  schedule_expression = format(
    "cron(%s %s ? * %s *)",
    split(":", each.value.start_time)[1],
    split(":", each.value.start_time)[0],
    join(",", [for day in each.value.days : upper(substr(day, 0, 3))])
  )
}
```

## 8. Common Issues and Solutions

### 7.1 Undefined Variable Error
```hcl
# Error: Reference to undeclared input variable
variable "region" {
  type = string
  # Missing default value
}

# Solution: Add default value or ensure value is provided
variable "region" {
  type    = string
  default = "us-west-2"
}
```

### 7.2 Type Constraints
```hcl
# Error: Invalid value for variable
variable "ports" {
  type    = list(number)
  default = ["80", "443"]  # String values in number list
}

# Solution: Use correct types
variable "ports" {
  type    = list(number)
  default = [80, 443]
}
```

### 7.3 Complex Type Validation
```hcl
variable "instance_config" {
  type = object({
    instance_type = string
    volume_size   = number
  })
  
  validation {
    condition     = can(regex("^t[23].*", var.instance_config.instance_type))
    error_message = "Instance type must be t2 or t3 series."
  }
  
  validation {
    condition     = var.instance_config.volume_size >= 20
    error_message = "Volume size must be at least 20 GB."
  }
}
```
