# 1. The terraform providers command shows information about the provider requirements of the configuration in the current working directory. True or False?

True. The "terraform providers" command displays the providers needed by the configuration.

For example:

$ terraform providers

Providers required by configuration:
.
├── provider[registry.terraform.io/hashicorp/local]
└── provider[registry.terraform.io/hashicorp/aws]

---
# 2. Select the reasons why we may need to specify the provider's argument?

There are two reasons to use a provider argument in the configuration.

1. To override the default provider configuration. For example, the default configuration may be to deploy resources in the "us-east-1" region. If the requirement is to deploy resources in a different region, we can use the provider argument to override the default.

2. In some cases, a configuration may need to use multiple versions of the same provider. For example - a resource that deploys to the "us-east-1" and another resource within the same configuration that deploys to the "us-west-2" region.

---
