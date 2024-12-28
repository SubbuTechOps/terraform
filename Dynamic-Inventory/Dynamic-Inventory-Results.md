# Terraform Dynamic Inventory - Results After Apply

## 1. Count Example Results

### Code:
```hcl
resource "aws_instance" "server" {
  count         = 3
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "Server-${count.index + 1}"
  }
}
```

### Result After Apply:
```plaintext
Created Resources:
✓ Three EC2 instances with the following names:
  - Server-1
  - Server-2
  - Server-3

Terraform State:
aws_instance.server[0]:
  id = "i-1234567890abc0001"
  tags = {
    Name = "Server-1"
  }

aws_instance.server[1]:
  id = "i-1234567890abc0002"
  tags = {
    Name = "Server-2"
  }

aws_instance.server[2]:
  id = "i-1234567890abc0003"
  tags = {
    Name = "Server-3"
  }
```

## 2. For_Each Example Results

### Code:
```hcl
variable "servers" {
  default = {
    web = "t2.micro"
    app = "t2.small"
    db  = "t2.medium"
  }
}

resource "aws_instance" "server" {
  for_each      = var.servers
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = each.value
}
```

### Result After Apply:
```plaintext
Created Resources:
✓ Three EC2 instances with different instance types:
  - web: t2.micro instance
  - app: t2.small instance
  - db: t2.medium instance

Terraform State:
aws_instance.server["web"]:
  id = "i-1234567890abc0001"
  instance_type = "t2.micro"
  tags = {
    Name = "web"
  }

aws_instance.server["app"]:
  id = "i-1234567890abc0002"
  instance_type = "t2.small"
  tags = {
    Name = "app"
  }

aws_instance.server["db"]:
  id = "i-1234567890abc0003"
  instance_type = "t2.medium"
  tags = {
    Name = "db"
  }
```

## 3. Dynamic Block Results

### Code:
```hcl
variable "inbound_ports" {
  default = [80, 443, 22]
}

resource "aws_security_group" "web" {
  name = "web-sg"

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

### Result After Apply:
```plaintext
Created Resources:
✓ One security group with three ingress rules:

Terraform State:
aws_security_group.web:
  id = "sg-1234567890abc"
  name = "web-sg"
  ingress = [
    {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]
```

## 4. Subnet Creation Example Results

### Code:
```hcl
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

resource "aws_subnet" "main" {
  for_each          = var.subnets
  vpc_id            = aws_vpc.main.id
  cidr_block        = each.value.cidr
  availability_zone = each.value.az
}
```

### Result After Apply:
```plaintext
Created Resources:
✓ Three subnets with different configurations:

Terraform State:
aws_subnet.main["public1"]:
  id = "subnet-1234567890abc0001"
  vpc_id = "vpc-1234567890abc"
  cidr_block = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  tags = {
    Name = "public1"
    Type = "public"
  }

aws_subnet.main["public2"]:
  id = "subnet-1234567890abc0002"
  vpc_id = "vpc-1234567890abc"
  cidr_block = "10.0.2.0/24"
  availability_zone = "us-east-1b"
  tags = {
    Name = "public2"
    Type = "public"
  }

aws_subnet.main["private1"]:
  id = "subnet-1234567890abc0003"
  vpc_id = "vpc-1234567890abc"
  cidr_block = "10.0.3.0/24"
  availability_zone = "us-east-1a"
  tags = {
    Name = "private1"
    Type = "private"
  }
```

## 5. Security Group with Rules Example Results

### Code:
```hcl
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
      cidr_blocks = ["10.0.0.0/8"]
    }
  }
}

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
```

### Result After Apply:
```plaintext
Created Resources:
✓ One security group with three ingress rules:

Terraform State:
aws_security_group.web:
  id = "sg-1234567890abc"
  name = "web-sg"
  ingress = [
    {
      description = "http"
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      description = "https"
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      description = "ssh"
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = ["10.0.0.0/8"]
    }
  ]
```

## Key Points to Remember:

1. Count Results:
   - Resources are created with numeric index [0], [1], [2]
   - Good for identical resources
   - Reference as: aws_instance.server[0]

2. For_Each Results:
   - Resources are created with string keys
   - Better for unique configurations
   - Reference as: aws_instance.server["web"]

3. Dynamic Block Results:
   - Creates multiple blocks within a single resource
   - All blocks are part of the same resource
   - Good for repeated configurations

4. Important Notes:
   - Count uses numeric indexes (0,1,2...)
   - For_each uses string keys ("web", "app", "db")
   - Dynamic blocks create multiple configurations within one resource
   - State shows exactly what was created
