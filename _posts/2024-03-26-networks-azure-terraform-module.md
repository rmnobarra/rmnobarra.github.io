---
layout: post
title: "A terraform module for Azure Networks"
date: 2024-03-26 10:00:00
categories: [terraform, azure]
tags: [terraform, devops, iac, sre, azure]
---

## Introduction

Connectivity is crucial for any service that needs to connect with someone, and a resource present in all kinds of environments, whether on-premises or in the cloud, is the Virtual Network.

A virtual network functions similarly to a physical network, complete with OSI layers, protocols, subnets, and so on. The purpose of this post is to demonstrate how to deploy a virtual network and subnets in Azure Cloud using a module written by me. Why? Because writing Terraform modules is fun, and for enterprise companies, it can be beneficial to have full control over a module and its code.

![virtual networks](https://rmnobarradev.blob.core.windows.net/rmnobarradev/azure_network.png)

## The module

main.tf:

```hcl
resource "azurerm_virtual_network" "vnet" {
  name                = var.vnet_name
  address_space       = var.address_space
  location            = var.location
  resource_group_name = var.resource_group_name
}

resource "azurerm_subnet" "subnet" {
  count                = length(var.subnets)
  name                 = var.subnets[count.index]["name"]
  resource_group_name  = var.resource_group_name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = [var.subnets[count.index]["address_prefix"]]

  depends_on = [ azurerm_virtual_network.vnet ]
}
```

The first resource block, 'azurerm_virtual_network', creates a virtual network. It is necessary to specify the resource group name.

The second block, 'azurerm_subnet', utilizes the resource group from the virtual network created in the previous step.

This resource employs a trick by utilizing the 'count' built-in function to generate a range of subnets that are provided to us as variables. Below are more details about it.

variables.tf

```hcl
variable "vnet_name" {
  description = "Name of the virtual network."
  type        = string
}

variable "address_space" {
  description = "Address space of the virtual network."
  type        = list(string)
}

variable "location" {
  description = "Azure location where the virtual network will be created."
  type        = string
}

variable "resource_group_name" {
  description = "Name of the resource group."
  type        = string
}

variable "subnets" {
  description = "List of subnets to create."
  type        = list(map(string))
}
```

The trick relies on subnet variables, which receive a list of strings. When we execute this module, I promise it will become clearer.

outputs.tf

```hcl
output "vnet_id" {
  value = azurerm_virtual_network.vnet.id
}

output "vnet_name" {
  value = azurerm_virtual_network.vnet.name
}

output "subnet_ids" {
  value = { for s in azurerm_subnet.subnet : s.name => s.id }
}
```

Nothing new here. Just print the virtual network ID, name, and subnet IDs.

providers.tf

```hcl
terraform {
  required_version = ">=0.12"

  required_providers {

    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>2.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

And finally a provider block.

## Using module

main.tf

```hcl
module "network" {
  source    = "git::https://github.com/rmnobarra/azure-network-terraform-module.git?ref=v1.0.0"
  vnet_name           = "vnet"
  address_space       = ["10.11.0.0/16"]
  location            = "brazilsouth"
  resource_group_name = "develop"
  subnets = [
    {
      name           = "subnet_a"
      address_prefix = "10.11.1.0/24"
    },
    {
      name           = "subnet_b"
      address_prefix = "10.11.2.0/24"
    },
    {
      name           = "subnet_c"
      address_prefix = "10.11.3.0/24"
    }
  ]
}
```

The source line I pass to the URL module is a Git repository that contains all the .hcl files mentioned above.

The magic lies in the 'subnets' variable, which makes subnet creation dynamic by sending a map to the module variable. Beautiful, don't you think?

outputs.tf

```hcl
output "vnet_id" {
  value = module.network.vnet_id
}

output "vnet_name" {
  value = module.network.vnet_name
}

output "subnet_ids" {
  value = module.network.subnet_ids
}
```

And the outputs block, thats print virtual network id, virtual network name and subnets id.

## Run module

After run `terraform plan` command the output below is expected:

```bash
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.network.azurerm_subnet.subnet[0] will be created
  + resource "azurerm_subnet" "subnet" {
      + address_prefix                                 = (known after apply)
      + address_prefixes                               = [
          + "10.11.1.0/24",
        ]
      + enforce_private_link_endpoint_network_policies = false
      + enforce_private_link_service_network_policies  = false
      + id                                             = (known after apply)
      + name                                           = "subnet_a"
      + resource_group_name                            = "develop"
      + virtual_network_name                           = "vnet"
    }

  # module.network.azurerm_subnet.subnet[1] will be created
  + resource "azurerm_subnet" "subnet" {
      + address_prefix                                 = (known after apply)
      + address_prefixes                               = [
          + "10.11.2.0/24",
        ]
      + enforce_private_link_endpoint_network_policies = false
      + enforce_private_link_service_network_policies  = false
      + id                                             = (known after apply)
      + name                                           = "subnet_b"
      + resource_group_name                            = "develop"
      + virtual_network_name                           = "vnet"
    }

  # module.network.azurerm_subnet.subnet[2] will be created
  + resource "azurerm_subnet" "subnet" {
      + address_prefix                                 = (known after apply)
      + address_prefixes                               = [
          + "10.11.3.0/24",
        ]
      + enforce_private_link_endpoint_network_policies = false
      + enforce_private_link_service_network_policies  = false
      + id                                             = (known after apply)
      + name                                           = "subnet_c"
      + resource_group_name                            = "develop"
      + virtual_network_name                           = "vnet"
    }

  # module.network.azurerm_virtual_network.vnet will be created
  + resource "azurerm_virtual_network" "vnet" {
      + address_space         = [
          + "10.11.0.0/16",
        ]
      + dns_servers           = (known after apply)
      + guid                  = (known after apply)
      + id                    = (known after apply)
      + location              = "brazilsouth"
      + name                  = "vnet"
      + resource_group_name   = "develop"
      + subnet                = (known after apply)
      + vm_protection_enabled = false
    }

Plan: 4 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + subnet_ids = {
      + subnet_a = (known after apply)
      + subnet_b = (known after apply)
      + subnet_c = (known after apply)
    }
  + vnet_id    = (known after apply)
  + vnet_name  = "vnet"
```

And virtual network and subnets created in Azure:

![virtual network azure](https://rmnobarradev.blob.core.windows.net/rmnobarradev/vnets.png)

## Conclusion

The Terraform module is very handy for sharing reusable infrastructure pieces, and turning those components into products is a great approach for DevOps teams to standardize deliveries. This, once again, emphasizes the importance of turning Infrastructure as Code (IaC) into tangible tech products.

## References

This modules lives in https://github.com/rmnobarra/azure-network-terraform-module, be my guest for any pull request.

## Any sugests or doubt? 

Feel free to reach out to me on social media: [twitter](https://twitter.com/rmnobarra)
,[linkedin](https://www.linkedin.com/in/rmnobarra/) and [github](https://github.com/rmnobarra).

You can also email me directly at rmnobarra@gmail.com. 

## Support

Did you really enjoy my content? Consider buying me a coffee through my Bitcoins wallets: 

![Donate with Bitcoin](https://img.shields.io/badge/Donate%20with-Bitcoin-orange)

Bitcoin Wallet:

`bc1quuv5hml9fjkf7azgwkt4xp867pzdwzyga33qmj`

![Bitcoin wallet QRCODE](https://rmnobarradev.blob.core.windows.net/rmnobarradev/bItcoin-address.png)

![Donate through Lightning Network](https://img.shields.io/badge/Donate%20with-Lighting-blue)

Lighting Address: 

`lnbc1pjue6mkpp5yj737e7fm6efhlj6sns42a875pmkencqmvdshf4ghlnntaet5llsdqqcqzzsxqrrsssp5d9hxl686w839qkwmkm0m30cf5gp4gnzxh68kss393xqzlsg0wr3q9qyyssqp3933zc3fg46nk3vafh63r3lqd0jn2p04w5xrz77h33rrr3xm7ckegg6s2ss64g4rf4jg87zdjzkl5tup7umqpghy2qnk65tqzsnplcpwv6z4c`

![Lighting wallet QRCODE](https://rmnobarradev.blob.core.windows.net/rmnobarradev/lighting-address.png)

Bye!