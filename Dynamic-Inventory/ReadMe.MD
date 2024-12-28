# Understanding Dynamic Inventory in Terraform

## 1. Basic Concepts

### What is Dynamic Inventory?
> 💡 Think of dynamic inventory as a way to create multiple similar resources without copying and pasting code.

### Two Main Ways to Create Multiple Resources:
1. Using `count` (Like a simple counter)
2. Using `for_each` (Like a detailed list)

## 2. Using Count - The Simple Way

### Basic Example: Creating Multiple EC2 Instances
```hcl
# Create three identical EC2 instances
resource "aws_instance" "server" {
  count         = 3                    # Creates 3 instances
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "Server-${count.index + 1}"  # Names will be Server-1, Server-2, Server-3
  }
}
```
> 📝 **Note**: `count.index` starts from 0, so we add 1 for human-readable names

### Slightly More Advanced Count Example
```hcl
# List of server names
variable "server_names" {
  type    = list(string)
  default = ["web", "app", "db"]
}

# Create instances with different names
resource "aws_instance" "server" {
  count         = length(var.server_names)        # Count based on list length
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = var.server_names[count.index]          # Use names from the list
  }
}
```
> ⚠️ **Warning**: If you remove an item from the middle of the list, it can affect other resources

## 3. Using For_Each - The Detailed Way

### Basic For_Each Example
```hcl
# Map of server configurations
variable "servers" {
  default = {
    web = "t2.micro"
    app = "t2.small"
    db  = "t2.medium"
  }
}

# Create instances with different sizes
resource "aws_instance" "server" {
  for_each      = var.servers
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = each.value  # Uses the instance type from the map

  tags = {
    Name = each.key    # Uses the key as the name (web, app, db)
  }
}
```
> 💡 **Note**: `each.key` is the map key (e.g., "web"), `each.value` is the corresponding value

## 4. Dynamic Blocks - For Resource Settings

### Simple Security Group Example
```hcl
# List of ports to open
variable "inbound_ports" {
  default = [80, 443, 22]
}

# Create security group with multiple rules
resource "aws_security_group" "web" {
  name = "web-sg"

  # Dynamic block for inbound rules
  dynamic "ingress" {
    for_each = var.inbound_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```
> 📝 **Note**: This creates three inbound rules, one for each port

### More Detailed Security Group Example
```hcl
# Detailed rule configurations
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
    ssh = {
      port        = 22
      cidr_blocks = ["10.0.0.0/8"]  # Internal network only
    }
  }
}

resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.sg_rules
    content {
      description = ingress.key           # Rule name (http, https, ssh)
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

## 5. Real-World Examples

### Example 1: Creating Multiple Subnets
```hcl
# Define subnet configurations
variable "subnets" {
  default = {
    public1 = {
      cidr = "10.0.1.0/24"
      az   = "us-east-1a"
      type = "public"
    }
    public2 = {
      cidr = "10.0.2.0/24"
      az   = "us-east-1b"
      type = "public"
    }
    private1 = {
      cidr = "10.0.3.0/24"
      az   = "us-east-1a"
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
```
> 💡 **Explanation**: 
> - `for_each` creates a subnet for each entry in the map
> - `each.key` is the subnet name (public1, public2, private1)
> - `each.value` contains the configuration (cidr, az, type)

### Example 2: Creating EC2 Instances with Different Configurations
```hcl
# Define instance configurations
variable "instances" {
  default = {
    "web-prod" = {
      type = "t2.medium"
      env  = "production"
      disk = 100
    }
    "web-dev" = {
      type = "t2.micro"
      env  = "development"
      disk = 20
    }
  }
}

# Create EC2 instances
resource "aws_instance" "servers" {
  for_each      = var.instances
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = each.value.type

  root_block_device {
    volume_size = each.value.disk
  }

  tags = {
    Name        = each.key
    Environment = each.value.env
  }
}
```
> 📝 **Note**: This creates two instances with different:
> - Instance types
> - Disk sizes
> - Environment tags

## 6. When to Use What

### Use Count When:
- Creating multiple identical resources
- Only the name or index needs to be different
- Simple numbering is sufficient

### Use For_Each When:
- Resources need different configurations
- You need to manage resources by name
- You want to avoid issues with list reordering

### Use Dynamic Blocks When:
- You have repeated configuration blocks within a resource
- You need to create multiple similar settings
- You want to make configurations more maintainable

## 7. Common Patterns and Tips

### 1. Combining Count with Conditional Creation
```hcl
# Create resources only in production
resource "aws_instance" "monitoring" {
  count = var.environment == "production" ? 2 : 0
  # ... rest of configuration
}
```

### 2. Using For_Each with Transformed Data
```hcl
locals {
  # Add common configuration to instances
  instance_config = {
    for name, config in var.instances : name => merge(config, {
      backup    = true
      monitored = true
    })
  }
}
```

### 3. Error Prevention
```hcl
# Validate input values
variable "instance_count" {
  type        = number
  description = "Number of instances to create"
  
  validation {
    condition     = var.instance_count > 0 && var.instance_count <= 5
    error_message = "Instance count must be between 1 and 5."
  }
}
```
