# The terraform providers command shows information about the provider requirements of the configuration in the current working directory. True or False?
True. The "terraform providers" command displays the providers needed by the configuration.

For example:

iac-server $ terraform providers

Providers required by configuration:
.
├── provider[registry.terraform.io/hashicorp/local]
└── provider[registry.terraform.io/hashicorp/aws]
