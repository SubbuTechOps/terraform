# Understanding Outputs with Dynamic Resources in Terraform

## 1. Outputs with Count

### Basic Count Example
```hcl
# Create multiple EC2 instances
resource "aws_instance" "server" {
  count         = 3
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "Server-${count.index + 1}"
  }
}

# Output all instance IDs as a list
output "server_ids" {
  value       = aws_instance.server[*].id
  description = "IDs of all created instances"
}

# Output specific instance ID
output "first_server_id" {
  value       = aws_instance.server[0].id
  description = "ID of the first instance"
}

# Output all instance private IPs
output "server_private_ips" {
  value       = aws_instance.server[*].private_ip
  description = "Private IPs of all instances"
}
```
> 📝 **Note**: The [*] syntax is used to get a list of all values from counted resources

## 2. Outputs with For_Each

### Basic For_Each Example
```hcl
# Define server configurations
variable "servers" {
  default = {
    web = "t2.micro"
    app = "t2.small"
    db  = "t2.medium"
  }
}

# Create instances
resource "aws_instance" "server" {
  for_each      = var.servers
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = each.value

  tags = {
    Name = each.key
  }
}

# Output all instance IDs as a map
output "server_ids" {
  value = {
    for k, v in aws_instance.server : k => v.id
  }
  description = "Map of server names to instance IDs"
}

# Output specific server ID
output "web_server_id" {
  value       = aws_instance.server["web"].id
  description = "ID of the web server"
}

# Output all private IPs with custom formatting
output "server_ips" {
  value = {
    for k, v in aws_instance.server : k => {
      private_ip = v.private_ip
      public_ip  = v.public_ip
    }
  }
  description = "Map of server IPs"
}
```
> 💡 **Note**: For resources created with for_each, you reference them using map syntax

## 3. Complex Output Examples

### Example 1: Subnet Outputs
```hcl
# Subnet configurations
variable "subnets" {
  default = {
    public1 = {
      cidr = "10.0.1.0/24"
      az   = "us-east-1a"
      type = "public"
    }
    private1 = {
      cidr = "10.0.2.0/24"
      az   = "us-east-1b"
      type = "private"
    }
  }
}

# Create subnets
resource "aws_subnet" "main" {
  for_each          = var.subnets
  vpc_id            = aws_vpc.main.id
  cidr_block        = each.value.cidr
  availability_zone = each.value.az

  tags = {
    Name = each.key
    Type = each.value.type
  }
}

# Output subnet information
output "subnet_info" {
  value = {
    for name, subnet in aws_subnet.main : name => {
      id         = subnet.id
      cidr_block = subnet.cidr_block
      az         = subnet.availability_zone
      type       = subnet.tags["Type"]
    }
  }
  description = "Detailed information about created subnets"
}

# Output only public subnet IDs
output "public_subnet_ids" {
  value = {
    for name, subnet in aws_subnet.main :
    name => subnet.id
    if subnet.tags["Type"] == "public"
  }
  description = "IDs of public subnets only"
}
```

### Example 2: Security Group with Dynamic Blocks and Outputs
```hcl
# Security group rules
variable "sg_rules" {
  default = {
    http = {
      port        = 80
      cidr_blocks = ["0.0.0.0/0"]
    }
    https = {
      port        = 443
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}

# Create security group
resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.sg_rules
    content {
      description = ingress.key
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}

# Output security group details
output "security_group_config" {
  value = {
    id = aws_security_group.web.id
    rules = {
      for rule in aws_security_group.web.ingress : rule.description => {
        port        = rule.from_port
        cidr_blocks = rule.cidr_blocks
      }
    }
  }
  description = "Security group configuration details"
}
```

## 4. Using Outputs in Other Resources

### Example: Using Dynamic Resource Outputs
```hcl
# Create EC2 instances in multiple subnets
resource "aws_instance" "app" {
  for_each = aws_subnet.main

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  subnet_id     = each.value.id  # Reference subnet ID

  tags = {
    Name = "App-${each.key}"
  }
}

# Output instance details with subnet information
output "instance_details" {
  value = {
    for name, instance in aws_instance.app : name => {
      instance_id = instance.id
      subnet_id   = instance.subnet_id
      private_ip  = instance.private_ip
      subnet_name = aws_subnet.main[name].tags["Name"]
    }
  }
  description = "Detailed instance information including subnet details"
}
```

## 5. Formatting Output Data

### Example: Custom Output Formatting
```hcl
# Create formatted output for documentation
output "infrastructure_details" {
  value = templatefile("${path.module}/templates/output.tpl", {
    instances = {
      for name, instance in aws_instance.server : name => {
        id         = instance.id
        private_ip = instance.private_ip
        type       = instance.instance_type
      }
    }
    subnets = {
      for name, subnet in aws_subnet.main : name => {
        id         = subnet.id
        cidr_block = subnet.cidr_block
      }
    }
  })
  description = "Formatted infrastructure details"
}

# Output sensitive information
output "instance_passwords" {
  value = {
    for name, instance in aws_instance.server : name => nonsensitive(instance.password)
  }
  sensitive = true  # Marks output as sensitive
  description = "Instance passwords (sensitive)"
}
```

## 6. Common Output Patterns

### Pattern 1: Filtering Outputs
```hcl
# Output only production instances
output "prod_instances" {
  value = {
    for name, instance in aws_instance.server :
    name => instance.id
    if contains(instance.tags, "Production")
  }
}
```

### Pattern 2: Transforming Outputs
```hcl
# Transform output format
output "instance_summary" {
  value = [
    for name, instance in aws_instance.server : {
      name       = name
      id         = instance.id
      private_ip = instance.private_ip
    }
  ]
}
```

### Pattern 3: Combining Multiple Resources
```hcl
# Output combined resource information
output "environment_summary" {
  value = {
    instances = {
      for name, instance in aws_instance.server : name => instance.id
    }
    subnets = {
      for name, subnet in aws_subnet.main : name => subnet.id
    }
    security_groups = [aws_security_group.web.id]
  }
}
```
