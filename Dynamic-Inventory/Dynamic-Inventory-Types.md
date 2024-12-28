# Quick Notes: Terraform Dynamic Inventory Types

## 1. Count Method 🔢
> Simple way to create multiple identical resources

```hcl
resource "aws_instance" "server" {
  count = 3
  # ... configuration
}
```

### Key Points:
- ✅ Good for creating multiple identical resources
- ✅ Resources are indexed numerically [0], [1], [2]
- ✅ Simple to understand and implement
- ⚠️ Can cause issues if you remove items from middle of list
- 🎯 Best for: When you need exact number of identical resources

### When to Use:
- Creating multiple identical EC2 instances
- Multiple identical subnets
- Any scenario where resources are exactly the same

### When Not to Use:
- Resources need different configurations
- Need to maintain specific resource names
- Order of resources matters

## 2. For_Each Method 🔄
> Create multiple resources with unique configurations

```hcl
resource "aws_instance" "server" {
  for_each = {
    web = "t2.micro"
    app = "t2.small"
  }
  instance_type = each.value
}
```

### Key Points:
- ✅ Creates resources with unique names/configurations
- ✅ Uses map keys instead of numeric indices
- ✅ More stable when removing/adding resources
- ✅ Better for managing individual resources
- 🎯 Best for: Resources with different configurations

### When to Use:
- Different instance types per server
- Multiple subnets with different CIDR blocks
- Resources that need unique names/tags

### When Not to Use:
- All resources are identical
- Simple numeric scaling is needed
- Just need a count of resources

## 3. Dynamic Blocks 📦
> Create multiple similar blocks within a single resource

```hcl
resource "aws_security_group" "example" {
  dynamic "ingress" {
    for_each = var.ports
    content {
      port = ingress.value
    }
  }
}
```

### Key Points:
- ✅ Creates multiple similar blocks in one resource
- ✅ Reduces code repetition
- ✅ Good for configurable resource settings
- 🎯 Best for: Multiple similar configurations within a single resource

### When to Use:
- Security group rules
- IAM policy statements
- Multiple similar configurations in one resource

### When Not to Use:
- Creating separate resources
- Simple configurations
- When clarity is more important than DRY code

## 4. Common Use Cases & Examples

### Count Example:
```hcl
# Creating multiple identical instances
resource "aws_instance" "web" {
  count = 3
  ami   = "ami-123"
  tags  = {
    Name = "web-${count.index + 1}"
  }
}

# Result:
# web-1, web-2, web-3
```

### For_Each Example:
```hcl
# Creating instances with different sizes
resource "aws_instance" "servers" {
  for_each = {
    web = "t2.micro"
    app = "t2.medium"
    db  = "t2.large"
  }
  instance_type = each.value
  tags = {
    Name = each.key
  }
}

# Result:
# web: t2.micro
# app: t2.medium
# db: t2.large
```

### Dynamic Block Example:
```hcl
# Multiple security group rules
resource "aws_security_group" "web" {
  dynamic "ingress" {
    for_each = [80, 443, 8080]
    content {
      port = ingress.value
    }
  }
}

# Result:
# Rules for ports 80, 443, and 8080
```

## 5. Best Practices 🎯

### Count:
- Use for identical resources
- Keep index references simple
- Use when order doesn't matter

### For_Each:
- Use meaningful map keys
- Good for configuration variations
- Better for resource tracking

### Dynamic Blocks:
- Use for repeated nested blocks
- Keep logic simple
- Document the expected outcome

## 6. Quick Decision Guide 🤔

Use Count When:
- Need X number of identical things
- Simple scaling is required
- Resources are truly identical

Use For_Each When:
- Resources need unique names
- Different configurations needed
- Need stable resource addresses

Use Dynamic Blocks When:
- Repeated blocks within one resource
- Configurable resource settings
- Reducing repetitive code

## 7. Common Pitfalls to Avoid ⚠️

### Count:
- Avoid removing middle items
- Don't rely on index for critical configurations
- Don't use for different configurations

### For_Each:
- Don't use for simple numbering
- Ensure map keys are unique
- Don't overcomplicate keys

### Dynamic Blocks:
- Don't overuse for simple configs
- Keep content blocks simple
- Don't nest too deeply
