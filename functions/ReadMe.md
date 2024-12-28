# Terraform Functions Guide with Notes

> 📌 **Quick Reference**: Functions in Terraform help manipulate values, transform data, and perform calculations.

## Table of Contents
1. String Functions
2. Numeric Functions
3. Collection Functions
4. Type Conversion Functions
5. Encoding Functions
6. Date and Time Functions
7. IP Network Functions
8. File System Functions

## 1. String Functions
> 🔤 **Purpose**: Manipulate and format text strings in your configurations

### lower & upper
```hcl
# Converts string case
local.name = lower("HELLO")    # Result: "hello"
local.name = upper("hello")    # Result: "HELLO"
```
> 📝 **Note**: Useful for ensuring consistent naming conventions in resources

### substr
```hcl
# Extract part of a string (starting position, length)
local.prefix = substr("terraform", 0, 4)  # Result: "terr"
```
> 📝 **Note**: Good for extracting specific parts of strings like prefixes or suffixes

### format
```hcl
# Create formatted string with placeholders
local.name = format("app-%s-%s", var.env, var.region)
# Example: var.env="prod", var.region="us-east-1"
# Result: "app-prod-us-east-1"
```
> 📝 **Note**: Essential for creating standardized resource names

### replace
```hcl
# Replace characters in a string
local.clean_name = replace(var.name, "/[^a-zA-Z0-9]/", "-")
# Example: var.name="my@app" → "my-app"
```
> 📝 **Note**: Helps ensure names comply with resource naming requirements

## 2. Numeric Functions
> 🔢 **Purpose**: Perform mathematical operations and number manipulations

### max & min
```hcl
local.max_value = max(5, 12, 9)    # Result: 12
local.min_value = min(5, 12, 9)    # Result: 5
```
> 📝 **Note**: Useful for determining resource sizes or counts

### ceil & floor
```hcl
local.ceiling = ceil(10.1)    # Result: 11
local.floor = floor(10.9)     # Result: 10
```
> 📝 **Note**: Helpful when dealing with resource calculations that need whole numbers

### abs
```hcl
# Get absolute value
local.absolute = abs(-42)     # Result: 42
```
> 📝 **Note**: Ensures positive values in calculations

## 3. Collection Functions
> 📚 **Purpose**: Handle lists, maps, and sets of values

### length
```hcl
# Get size of a collection
local.subnet_count = length(var.subnet_cidrs)
```
> 📝 **Note**: Essential for dynamic resource creation based on list sizes

### merge
```hcl
# Combine multiple maps
locals {
  tags = merge(
    var.common_tags,      # Base tags
    {
      Environment = var.env  # Additional tags
    }
  )
}
```
> 📝 **Note**: Critical for combining different sets of tags or properties

### concat
```hcl
# Join multiple lists
local.all_users = concat(var.admin_users, var.regular_users)
```
> 📝 **Note**: Useful for combining multiple lists of similar items

### keys & values
```hcl
# Extract keys or values from a map
local.tag_keys = keys(var.tags)
local.tag_values = values(var.tags)
```
> 📝 **Note**: Helpful when you need to work with map components separately

## 4. Type Conversion Functions
> 🔄 **Purpose**: Convert between different data types

### tostring
```hcl
# Convert value to string
local.port_string = tostring(80)  # Result: "80"
```
> 📝 **Note**: Useful when you need string representation of numbers

### tonumber
```hcl
# Convert string to number
local.count = tonumber("42")  # Result: 42
```
> 📝 **Note**: Important when working with number inputs provided as strings

### tolist & toset
```hcl
# Convert to list or set
local.unique_values = toset(var.items)  # Removes duplicates
```
> 📝 **Note**: toset is particularly useful for ensuring unique values

## 5. Encoding Functions
> 🔐 **Purpose**: Handle data encoding and decoding

### base64encode & base64decode
```hcl
# Encode/decode base64
local.encoded = base64encode("hello")  # For user data scripts
local.decoded = base64decode(local.encoded)
```
> 📝 **Note**: Commonly used for user data in EC2 instances

### jsonencode & jsondecode
```hcl
# Convert to/from JSON
local.json_data = jsonencode({
  name = "app"
  port = 80
})
```
> 📝 **Note**: Essential for working with APIs or policy documents

## 6. Date and Time Functions
> ⏰ **Purpose**: Handle date and time operations

### timestamp
```hcl
# Get current time
local.current_time = timestamp()
# Result: "2024-12-29T10:15:30Z"
```
> 📝 **Note**: Useful for tagging resources with creation time

### formatdate
```hcl
# Format date strings
local.formatted_date = formatdate("YYYY-MM-DD", timestamp())
# Result: "2024-12-29"
```
> 📝 **Note**: Helps create standardized date formats for tags or names

## 7. IP Network Functions
> 🌐 **Purpose**: Calculate IP addresses and ranges

### cidrsubnet
```hcl
# Calculate subnet addresses
local.subnet = cidrsubnet("10.0.0.0/16", 8, 1)
# Result: "10.0.1.0/24"
```
> 📝 **Note**: Essential for VPC and subnet design

### cidrhost
```hcl
# Calculate specific IP in CIDR range
local.ip = cidrhost("10.0.0.0/24", 5)
# Result: "10.0.0.5"
```
> 📝 **Note**: Useful for static IP assignment within subnets

## 8. File System Functions
> 📁 **Purpose**: Work with files in your Terraform configurations

### file
```hcl
# Read file content
local.script = file("${path.module}/script.sh")
```
> 📝 **Note**: Commonly used for reading scripts or config files

### fileexists
```hcl
# Check for file existence
local.exists = fileexists("${path.module}/config.json")
```
> 📝 **Note**: Good for conditional file operations

## Common Use Cases

### 1. Resource Naming Pattern
```hcl
locals {
  # Create standardized name
  name = lower(format("%s-%s-%s", 
    var.project,
    var.env,
    var.component
  ))
}
```
> 📝 **Note**: Ensures consistent resource naming across your infrastructure

### 2. Tag Management
```hcl
locals {
  # Combine and standardize tags
  required_tags = merge(
    var.common_tags,
    {
      Environment = var.env
      CreatedAt   = formatdate("YYYY-MM-DD", timestamp())
    }
  )
}
```
> 📝 **Note**: Creates consistent tagging strategy across resources

### 3. Network Calculations
```hcl
# Create multiple subnets automatically
resource "aws_subnet" "private" {
  count             = 3
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone = var.availability_zones[count.index]
}
```
> 📝 **Note**: Automates subnet creation with proper CIDR ranges

## Best Practices

1. Use locals for complex calculations
```hcl
locals {
  name_prefix = lower(format("%s-%s", var.project, var.env))
}
```
> 📝 **Note**: Makes code more readable and reusable

2. Handle null values safely
```hcl
locals {
  tags = coalesce(var.custom_tags, {})
}
```
> 📝 **Note**: Prevents errors from null values

3. Standardize string formatting
```hcl
locals {
  name = lower(replace(var.name, "/[^a-zA-Z0-9-]/", "-"))
}
```
> 📝 **Note**: Ensures consistent naming conventions
