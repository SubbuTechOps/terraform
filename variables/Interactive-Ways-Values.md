# Interactive Ways to Pass Values to Terraform Variables

## 1. Variable Prompts

### Basic Variable Declaration (without default)
```hcl
# variables.tf
variable "environment" {
  type        = string
  description = "Environment name (dev/staging/prod)"
}
```
When you run `terraform plan` or `terraform apply`, Terraform will interactively prompt for the value:
```bash
var.environment
  Environment name (dev/staging/prod)
  Enter a value: 
```

### Variables with Validation
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
If invalid input is provided:
```bash
var.environment
  Environment name
  Enter a value: test

Error: Invalid value for variable
Environment must be dev, staging, or prod.
```

## 2. Interactive Environment Variables
```bash
# Interactive shell prompt
read -s TF_VAR_db_password
export TF_VAR_db_password
```

## 3. Using terraform console for Interactive Testing

```bash
# Start terraform console
$ terraform console

# Test variable values
> var.environment
"production"

# Test expressions
> coalesce(var.custom_name, local.default_name)
```

## 4. Variable Handling Methods

### Method 1: Using -var with prompt
```bash
# Script to prompt and apply
#!/bin/bash
echo "Enter environment (dev/staging/prod):"
read env
terraform apply -var="environment=$env"
```

### Method 2: Interactive tfvars generation
```bash
# interactive-vars.sh
#!/bin/bash
echo "Generating tfvars file interactively"

# Prompt for values
echo "Enter environment (dev/staging/prod):"
read environment

echo "Enter instance type:"
read instance_type

# Create tfvars file
cat << EOF > terraform.tfvars
environment = "${environment}"
instance_type = "${instance_type}"
EOF

# Run Terraform
terraform apply
```

### Method 3: Using read command for sensitive data
```bash
#!/bin/bash
# Prompt for sensitive information
echo "Enter database password:"
read -s db_password

# Export as environment variable
export TF_VAR_db_password="$db_password"

# Run Terraform
terraform apply
```

## 5. Interactive Variable Input Scenarios

### Scenario 1: Basic Resource Creation
```hcl
# variables.tf
variable "instance_name" {
  type        = string
  description = "Name of the EC2 instance"
}

variable "instance_type" {
  type        = string
  description = "EC2 instance type"
}

# main.tf
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
  
  tags = {
    Name = var.instance_name
  }
}
```

Interactive script:
```bash
#!/bin/bash
echo "EC2 Instance Creation Wizard"

echo "Enter instance name:"
read instance_name

echo "Select instance type:"
echo "1) t2.micro"
echo "2) t2.small"
echo "3) t2.medium"
read -p "Choose (1-3): " choice

case $choice in
  1) instance_type="t2.micro" ;;
  2) instance_type="t2.small" ;;
  3) instance_type="t2.medium" ;;
  *) echo "Invalid choice"; exit 1 ;;
esac

terraform apply \
  -var="instance_name=$instance_name" \
  -var="instance_type=$instance_type"
```

### Scenario 2: Complex Configuration
```hcl
# variables.tf
variable "vpc_config" {
  type = object({
    name = string
    cidr = string
    public_subnets = number
    private_subnets = number
  })
  description = "VPC Configuration"
}
```

Interactive script:
```bash
#!/bin/bash
echo "VPC Configuration Wizard"

echo "Enter VPC name:"
read vpc_name

echo "Enter VPC CIDR (e.g., 10.0.0.0/16):"
read vpc_cidr

echo "Number of public subnets:"
read pub_subnets

echo "Number of private subnets:"
read priv_subnets

# Create JSON-formatted variable value
vpc_config="{
  \"name\": \"$vpc_name\",
  \"cidr\": \"$vpc_cidr\",
  \"public_subnets\": $pub_subnets,
  \"private_subnets\": $priv_subnets
}"

terraform apply -var="vpc_config=$vpc_config"
```

## 6. Best Practices for Interactive Input

1. Input Validation
```bash
#!/bin/bash
while true; do
    echo "Enter environment (dev/staging/prod):"
    read environment
    if [[ "$environment" =~ ^(dev|staging|prod)$ ]]; then
        break
    else
        echo "Invalid environment. Please try again."
    fi
done
```

2. Secure Password Input
```bash
#!/bin/bash
while true; do
    echo "Enter database password:"
    read -s password1
    echo "Confirm password:"
    read -s password2
    
    if [ "$password1" = "$password2" ]; then
        export TF_VAR_db_password="$password1"
        break
    else
        echo "Passwords don't match. Please try again."
    fi
done
```

3. Default Values
```bash
#!/bin/bash
echo "Enter instance type (default: t2.micro):"
read instance_type
instance_type=${instance_type:-t2.micro}

terraform apply -var="instance_type=$instance_type"
```

4. Save Inputs for Future Use
```bash
#!/bin/bash
echo "Would you like to save these values for future use? (y/n)"
read save_choice

if [ "$save_choice" = "y" ]; then
    cat << EOF > custom.auto.tfvars
environment = "${environment}"
instance_type = "${instance_type}"
EOF
    echo "Values saved to custom.auto.tfvars"
fi
```
