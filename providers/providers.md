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

**1. To override the default provider configuration.** For example, the default configuration may be to deploy resources in the "us-east-1" region. If the requirement is to deploy resources in a different region, we can use the provider argument to override the default.

**2. In some cases, a configuration may need to use multiple versions of the same provider.** For example - a resource that deploys to the "us-east-1" and another resource within the same configuration that deploys to the "us-west-2" region.

---

# 3. Choose the easiest way to list the versions of all installed plugins in terraform along with terraform versions.

Run **terraform version** command
---

# 4. Your team has deployed an EKS cluster in the AWS cloud using terraform. To the existing configuration, you have added a new resource block for the "kubernetes_deployment" type resource. When you run terraform apply, you see an error that states - “Failed to instantiate provider”. What could be the reson for this error?

**The kubernetes provider was not initalized for the configuration.**

Since the EKS cluster was already provisioned and the error was displayed only after adding the resource block for the "kubernetes_deployment", it would appear that a "terraform init" command was not run to download the provider plugin for the kubernetes provider.
