---
layout: post
title: "How to upload files to a public Azure blob storage with Terraform + Terraform public url's outputs"
date: 2023-12-13 11:44:00
categories: [azure, terraform, IaC]
tags: [azure, terraform, IaC, devops]
---

## Introduction

Recently, I needed to manage public files in a blob storage and initially thought of using Terraform to simplify the process. The documentation wasn't very helpful, and it took some effort to achieve my goal. In this post, I'll try to detail how I did it.

In time, Azure storage account is a highly available, secure, and massively scalable way to store any kind of data in Azure Cloud.

![upload cloud azure](https://rmnobarradev.blob.core.windows.net/rmnobarradev/upload-cloud-resized.png)

## Goals

A fully Azure Terraform project that creates:

* Resource group
* Storage account
* Storage container
* Upload files
* Output for uploaded files

## Provider and Azure Resource Group Blocks

These blocks manage a fundamental structure for our project:

```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rmnobarradev" {
  name     = "rmnobarradev"
  location = "West US 2"
}
```

## Storage Account and Storage Container Resource Blocks

Here lies the storage account structure:

```hcl
resource "azurerm_storage_account" "rmnobarradev" {
  name                     = "rmnobarradev"
  resource_group_name      = azurerm_resource_group.rmnobarradev.name
  location                 = azurerm_resource_group.rmnobarradev.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

resource "azurerm_storage_container" "container" {
  name                  = "rmnobarradev"
  storage_account_name  = azurerm_storage_account.rmnobarradev.name
  container_access_type = "blob"
}
```

## Storage blob and output blocks

Things get interesting here, with a for_each loop to interact with N files inside the "files" folder in the current directory (or any other folder that keep your files), and a another for loop in the output block to list all public URLs for our files.

```hcl
resource "azurerm_storage_blob" "upload_files" {
  for_each = fileset("./files", "**/*")

  name                   = each.value
  storage_account_name   = azurerm_storage_account.rmnobarradev.name
  storage_container_name = azurerm_storage_container.container.name
  type                   = "Block"
  source                 = "./files/${each.value}"
}


output "file_urls" {
  value = [for file in azurerm_storage_blob.upload_files : "${azurerm_storage_account.rmnobarradev.primary_blob_endpoint}${azurerm_storage_container.container.name}/${file.name}"]
}
```

The full block looks like:

```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rmnobarradev" {
  name     = "rmnobarradev"
  location = "West US 2"
}

resource "azurerm_storage_account" "rmnobarradev" {
  name                     = "rmnobarradev"
  resource_group_name      = azurerm_resource_group.rmnobarradev.name
  location                 = azurerm_resource_group.rmnobarradev.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

resource "azurerm_storage_container" "container" {
  name                  = "rmnobarradev"
  storage_account_name  = azurerm_storage_account.rmnobarradev.name
  container_access_type = "blob"
}

resource "azurerm_storage_blob" "upload_files" {
  for_each = fileset("./files", "**/*")

  name                   = each.value
  storage_account_name   = azurerm_storage_account.rmnobarradev.name
  storage_container_name = azurerm_storage_container.container.name
  type                   = "Block"
  source                 = "./files/${each.value}"
}

output "file_urls" {
  value = [for file in azurerm_storage_blob.upload_files : "${azurerm_storage_account.rmnobarradev.primary_blob_endpoint}${azurerm_storage_container.container.name}/${file.name}"]
}

```

## Conclusion

Terraform is an incredibly handy tool for solving many daily tasks. Using it with a loop structure enhances its utility. This relatively simple project demonstrates an elegant and quick way to use Terraform.

## Any sugests or doubt? 

Feel free to reach out to me on social media: [twitter](https://twitter.com/rmnobarra)
,[linkedin](https://www.linkedin.com/in/rmnobarra/) and [github](https://github.com/rmnobarra).

You can also email me directly at rmnobarra@gmail.com. 

## Support

Did you really enjoy my content? Consider buying me a coffee through my Bitcoin wallet: 

Bitcoin Wallet: `3GzvpHGd22HDbpnaZjU57jFT6ptXqNrGmT`

[![Donate with Bitcoin](https://img.shields.io/badge/Doar%20com-Bitcoin-orange)](bitcoin:3GzvpHGd22HDbpnaZjU57jFT6ptXqNrGmT)

Bye!