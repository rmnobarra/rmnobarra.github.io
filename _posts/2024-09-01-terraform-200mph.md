---
layout: post
title: "Terraform 200mph"
date: 2024-09-01 10:00:00
categories: [terraform]
tags: [terraform]
---

## Introduction

Terraform is an open-source tool developed by HashiCorp, written in Go, and distributed as a single binary for the main operating systems. This binary, simply called terraform, can be run directly on any machine, whether it's a personal laptop, a CI/CD server, or any other environment, without the need to install or manage additional infrastructure for its execution.

There are many tutorials and documentation on the internet—some easy, some difficult, and others too complex to grasp.

The main goal of this article is to demystify Terraform because:

- I believe this tool should be easy to use and understand. If it’s not, maybe something isn’t right.
- Terraform is AWESOME!

In this article, I’ve included 10 topics that, if you understand each one, Terraform will no longer seem complex to you.

The topics are:

1. [How Terraform Works](#1-how-terraform-works)
2. [Documentation](#2-documentation)
3. [HCL Language](#3-hcl-language)
4. [Terraform State](#4-terraform-state)
5. [Remote State](#5-remote-state-with-remote-backends)
6. [Workspaces](#6-workspaces)
7. [Variables](#7-variables)
8. [Style Guide](#8-style-guide)
9. [Loops](#9-loops)
10. [Modules](#10-modules)

Let' go!

![terraform 200mph](https://rmnobarradev.blob.core.windows.net/rmnobarradev/terraform-200mph.png)

## 1. How Terraform Works

Behind the scenes, Terraform makes API calls to cloud providers such as AWS, Azure, Google Cloud, DigitalOcean, OpenStack, among others. These calls allow Terraform to interact directly with the existing infrastructure in these providers, leveraging native authentication mechanisms like API keys, IAM permissions, or credential-based authentication that you already have in place.

Terraform operates in a declarative manner, meaning you define the desired state of the infrastructure, and it automatically calculates the necessary actions to reach that state, whether creating, updating, or destroying resources. It does this using a simple lifecycle, with the main commands being:

* **terraform init:** Initializes the working directory, downloading the necessary providers.
* **terraform plan:** Generates an execution plan showing the changes that will be applied.
* **terraform apply:** Executes the changes to align the infrastructure to the desired state.
* **terraform destroy:** Removes all managed resources, reverting the infrastructure to its initial state.

This simplicity, combined with its ability to interact with multiple providers transparently, makes Terraform a powerful tool for managing infrastructures in a unified, predictable, and efficient manner.

## 2. Documentation

The official Terraform documentation is not just a list of commands and options. It also provides examples, best practices, and a detailed view of each provider's resources, making it an invaluable tool for implementing IaC efficiently and securely.

The documentation goes beyond a simple list of commands and options. Instead, it offers:

* **Quick Start Guides:** Step-by-step instructions to quickly get started with Terraform.
* **Complete Command Reference:** Detailed information about each command, including options and flags.
* **Practical Examples:** Real-world use cases demonstrating how to implement various resources across different cloud providers, such as AWS, Azure, and Google Cloud.

## 3. HCL Language

The primary purpose of the HCL language is to declare infrastructure in a declarative way, and one or more `.tf` files, which form a Terraform project, serve to instruct the Terraform binary on how to manage the declared infrastructure.

Its syntax consists of a few basic elements:

```hcl
resource "aws_vpc" "main" {
  cidr_block = var.base_cidr_block
}

<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
  # Block body
  <IDENTIFIER> = <EXPRESSION> # Argument
}
```

Let's look at a practical example. To create an Nginx container using Terraform, you can use the Docker provider:

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "3.0.2"
    }
  }
}

provider "docker" {
  host = "unix:///var/run/docker.sock"
}

resource "docker_image" "nginx" {
  name = "nginx:latest"
}

resource "docker_container" "nginx_container" {
  image = docker_image.nginx.image_id
  name  = "nginx_container"

  ports {
    internal = 80
    external = 80
  }
}
```

* **Block:** A block in Terraform is a configuration unit that defines a resource, provider, or module. In the example above, we have three blocks: `provider`, `resource "docker_image"`, and `resource "docker_container"`.

* **Arguments:** Arguments are used within blocks to configure resources. They assign values to identifiers. For example, in the `provider "docker"` block, the `host` argument defines the path to the Docker socket.

* **Attributes:** Attributes are properties of resources that can be configured or referenced. In the block `resource "docker_container" "nginx"`, `image` and `name` are attributes of the `docker_container` resource.

* **Reference:** References are used to access attributes from other resources. In the example, `docker_image.nginx.latest` is a reference to the `latest` attribute of the `docker_image` resource named `nginx`.

Example 2: Azure Storage Account

```hcl
terraform {
  required_version = ">=0.12"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>2.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~>3.1.0"
    }
  }
}

module "resource_group" {
  source   = "git::https://github.com/rmnobarra/azure-resource-groups-terraform-module.git?ref=v1.0.0"
  rg_name  = "terraform-200mph"
  location = "westus2"
}

provider "azurerm" {
  features {}
}

provider "random" {
}

resource "random_string" "terraform-200mph" {
  length  = 6
  special = false
  upper   = false
}

output "random_string" {
  value = random_string.terraform-200mph.result
}

resource "azurerm_storage_account" "terraform-200mph" {
  name                     = "terraform200mph${random_string.terraform-200mph.result}"
  resource_group_name      = "terraform-200mph"
  location                 = "brazilsouth"
  account_tier             = "Standard"
  account_replication_type = "LRS"

  tags = {
    environment = "production"
  }
}

resource "azurerm_storage_container" "dev" {
  name                  = "dev"
  storage_account_name  = azurerm_storage_account.terraform-200mph.name
  container_access_type = "private"
}

resource "azurerm_storage_container" "prod" {
  name                  = "prod"
  storage_account_name  = azurerm_storage_account.terraform-200mph.name
  container_access_type = "private"
}

output "storage_account_key" {
  value     = azurerm_storage_account.terraform-200mph.primary_access_key
  sensitive = true
}

output "storage_account_name" {
  value = azurerm_storage_account.terraform-200mph.name
}

output "storage_account_url" {
  value = azurerm_storage_account.terraform-200mph.primary_blob_endpoint
}

output "storage_container" {
  value = [
    azurerm_storage_container.dev.name,
    azurerm_storage_container.prod.name
  ]
}
```

Example 3: AWS s3 bucket

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "random_pet" "bucket_name" {
  length = 2
}

resource "aws_s3_bucket" "example" {
  bucket = "${random_pet.bucket_name.id}-bucket"

}
```

## 4. Terraform State

The **state** is a central component in Terraform's operation, responsible for tracking the managed infrastructure. Each time Terraform is executed, it stores information about the provisioned resources in a state file called `terraform.tfstate`. This file is generated in the directory where Terraform is run and contains a mapping between the resources defined in the configuration files and their real-world representations. The format of this file is a custom JSON, and it allows Terraform to know the current state of the infrastructure in order to determine what changes need to be made.

### Why is Terraform State Important?

The state serves as Terraform’s source of truth. Without it, the tool wouldn’t be able to identify which resources have already been created, modified, or destroyed. This allows Terraform to perform incremental operations, where it applies only the necessary changes instead of reprovisioning everything from scratch. Additionally, the state enables Terraform to execute a series of validations and change predictions before applying any operations, such as with the `terraform plan` command, which displays a preview of the changes to be made.

### Challenges in Using Terraform State

While local use of the state file works well for personal projects, collaborative and production projects require more robust approaches. This is because, in teams, managing the state file needs to be shared and properly controlled to avoid issues such as:

* **Shared Storage:** In a collaborative environment, all team members need to access the same state file to keep the infrastructure in sync. This requires the state file to be stored in a shared and accessible location for everyone.

* **Locking:** When multiple users attempt to modify the state simultaneously, a race condition can occur, where concurrent changes may corrupt the state file. Without a locking system, there's a risk of data loss or conflicts in infrastructure management.

* **Environment Isolation:** To prevent changes in development or test environments from affecting production, it's necessary to isolate the state files for each environment. This ensures that errors in one environment do not impact others.

Aqui está a tradução em formato markdown:

## 5. Remote State with Remote Backends

The use of remote backends in Terraform is an effective solution to address common issues associated with storing and managing state files locally. Remote backends allow the state file to be stored in a centralized and shared location, offering benefits such as eliminating manual errors, support for locking, and secret protection.

* **Elimination of Manual Errors:** When the remote backend is configured, Terraform automatically loads the state file from the remote backend whenever the `terraform plan` or `terraform apply` commands are executed. Similarly, after each `apply`, Terraform saves the updated state to the remote backend. This eliminates the need to manually move or share the state file among team members, significantly reducing the chances of human errors, such as accidental loss or corruption of the state file.

* **Support for Locking:** One of the biggest challenges in using Terraform in collaborative environments is ensuring that multiple people don’t make simultaneous changes to the infrastructure, which could corrupt the state. Most remote backends natively support locking, meaning that when running the `terraform apply` command, Terraform automatically acquires a lock on the state. If another user is applying changes at the same time, the second will have to wait for the lock to be released. To adjust the wait time, the `-lock-timeout=<TIME>` parameter can be used, such as `-lock-timeout=10m`, which instructs Terraform to wait up to 10 minutes for the lock to be released.

* **Secret Protection:** Another crucial benefit of remote backends is security. Most of them support encryption in transit and at rest, ensuring that the state file is protected during transmission and while stored. Additionally, many backends, such as Amazon S3, allow access permissions to be configured through IAM policies, ensuring that only authorized individuals can access the state and any secrets it contains.

Although Terraform does not yet natively support encrypting secrets directly within the state file, the use of remote backends significantly mitigates security concerns, as the state is no longer stored in plain text on a local disk.

### Main Examples of Remote Backends

Some of the most commonly used remote backends in Terraform include:

* **Amazon S3:** Integration with S3 to store the state and use of DynamoDB for locks.
* **Azure Storage:** State storage in Azure Storage accounts.
* **Google Cloud Storage:** Uses Google Cloud buckets to store the state.
* **Terraform Cloud and Enterprise:** Offer advanced features such as access control, policy management, and team support.

## Using Remote State

To enable remote state using S3 as the backend, we first need to create the S3 bucket, starting with the provider:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

Now s3 resource:

```hcl
resource "aws_s3_bucket" "terraform_state" {
  bucket = "terraform200mph"

  # Prevent accidental deletion of this S3 bucket
  lifecycle {
    prevent_destroy = true
  }
}
```

Versioning ensures that a new state file is created with each execution:

```hcl
resource "aws_s3_bucket_versioning" "enabled" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}
```

As well as encryption:

```hcl
resource "aws_s3_bucket_server_side_encryption_configuration" "default" {
  bucket = aws_s3_bucket.terraform_state.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```

Explicitly blocking public access:


```hcl
resource "aws_s3_bucket_public_access_block" "public_access" {
  bucket                  = aws_s3_bucket.terraform_state.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

Now, we will create a table in AWS DynamoDB to handle locking, ensuring the uniqueness of each execution:

```hcl
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-up-and-running-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

The complete file should look like:

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "terraform_state" {
  bucket = "terraform200mph"

  lifecycle {
    prevent_destroy = false
  }
}

resource "aws_s3_bucket_versioning" "enabled" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "default" {
  bucket = aws_s3_bucket.terraform_state.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "public_access" {
  bucket                  = aws_s3_bucket.terraform_state.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform200mph-lock"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

The remote backend in Terraform is a solution that addresses common issues related to storing state files locally, such as the possibility of manual errors, lack of file locking, and the security of secrets. By using remote backends, the state of the infrastructure managed by Terraform can be stored centrally and securely, facilitating team collaboration and avoiding conflicts.

## Benefits of Remote Backends

* **Elimination of Manual Errors:** Once a remote backend is configured, Terraform automatically loads the state file from that backend whenever you run the `plan` or `apply` commands, and saves the updated state after execution. This eliminates the need to manually move files, reducing the risk of data loss or corruption.
* **File Locking:** Remote backends natively support locking. This means that when you run `terraform apply`, Terraform automatically acquires a lock to prevent other processes from altering the state at the same time, avoiding conflicts. If another user is already making a change, Terraform will wait until the lock is released, or until the defined timeout is reached, using the `-lock-timeout=<TIME>` parameter.
* **Secret Security:** Most remote backends support encryption of the state file both in transit and at rest. This ensures that sensitive information, such as API keys and credentials, is protected from unauthorized access. Additionally, these backends often allow access policies to be configured to control who can view and modify the state file.

## Configuring a Remote Backend with S3

A common example of a remote backend is using an S3 bucket to store the state and a DynamoDB table for locking. Below is a code snippet for reference:

```hcl
provider "aws" {
  region = "us-east-1"
}

terraform {
  backend "s3" {
    bucket         = "terraform200mph"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform200mph-lock"
    encrypt        = true
  }
}

resource "aws_s3_bucket" "terraform_state" {
  bucket = "terraform200mph"

  lifecycle {
    prevent_destroy = false
  }
}

resource "aws_s3_bucket_versioning" "enabled" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "default" {
  bucket = aws_s3_bucket.terraform_state.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "public_access" {
  bucket                  = aws_s3_bucket.terraform_state.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform200mph-lock"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

This configuration instructs Terraform to use the S3 bucket and DynamoDB table for state storage and locking.

## Limitations of Remote Backends in Terraform

Although remote backends solve many issues when storing infrastructure state, they also present some limitations that should be considered. Below are two main challenges:

## "Chicken and Egg" Scenario in Backend Creation

One of the biggest challenges with remote backends occurs when attempting to use Terraform itself to create the backend where the state will be stored, such as an S3 bucket. This requires a two-step process:

* First, you need to write Terraform code to create the necessary resources (e.g., an S3 bucket and a DynamoDB table) and use a temporary local backend to apply that configuration.
* Then, you need to modify the code to add the remote backend configuration and run `terraform init` again to migrate the local state to the remote backend.

This process, while functional, can be somewhat inconvenient and needs to be reversed if you ever need to delete the backend. The upside is that, once configured, the same backend can be shared across multiple Terraform configurations, simplifying future uses.

## Backend Configuration Limitations

Another significant limitation is that the backend block in Terraform does not allow the use of variables or references. For example, the following code will not work:

```hcl
terraform {
  backend "s3" {
    bucket         = var.bucket
    region         = var.region
    dynamodb_table = var.dynamodb_table
    key            = "example/terraform.tfstate"
    encrypt        = true
  }
}
```

This means you need to manually insert the names of buckets, tables, and other configurations directly into the code of each module. This can lead to a lot of code duplication and the need to copy and paste configurations into each module, increasing the risk of errors. To mitigate this issue, one option is to use partial configurations, where parameters like the bucket name and region are passed through a configuration file (e.g., backend.hcl) and referenced in the command line when running terraform init:

```bash
terraform init -backend-config=backend.hcl
```

## Other Considerations

* **Manual Management:** When using multiple modules, you must carefully manage the paths and keys used in the state to ensure each module has its own state space, preventing one module's state from overwriting another.
  
* **Fixed Backend Version:** Since the backend configuration cannot be dynamically altered using variables, it's important to carefully define and document the parameters used for each environment and module to avoid inconsistencies.

## 6. Workspaces

Workspaces in Terraform are a feature that allows managing different states for the same infrastructure configuration. They are useful for separating environments, such as development, testing, and production, using the same code. Instead of creating multiple duplicated configurations for each environment, workspaces enable you to reuse the same codebase while maintaining separate states in an organized manner.

## Workspace Concept

By default, Terraform operates in a workspace called "default." When a new workspace is created, Terraform maintains a separate state file for that workspace, allowing different states to exist for the same infrastructure configuration. This is particularly useful for environments that share the same architecture but need separation to avoid interference between them. For example:

* In the "dev" workspace, you can test changes without affecting production.
* In the "prod" workspace, the state reflects the production infrastructure, keeping it isolated from experiments or temporary adjustments.

Each workspace has its own state file, allowing complete isolation of resources.

## Main Commands for Workspace Management

Terraform provides a simple set of commands to manage and interact with workspaces:

Create a new workspace:


```bash
terraform workspace new <nome-do-workspace>
```

Example:

```bash
terraform workspace new dev
terraform workspace new prod
```

This command creates new workspaces called "dev" and "prod" with a separate state file for each.


Select an existing workspace:

```bash
terraform workspace select <workspace-name>
```

Example:

```bash
terraform workspace select prod
```

This changes the active workspace to "prod," so the next Terraform executions will use the state associated with that workspace.

Now, run the following code:

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "3.0.2"
    }
  }
}

provider "docker" {
  host = "unix:///var/run/docker.sock"
}

# Pulls the image
resource "docker_image" "nginx" {
  name = "rmnobarra/nginx:green"
}

# Create a container
resource "docker_container" "nginx_container" {
  image = docker_image.nginx.image_id
  name  = "nginx_container-${terraform.workspace}"

  ports {
    internal = 80
    external = 8080
  }
}
```

Now, switch the current workspace to "dev":

```bash
terraform workspace select dev
```

Run the following code:

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "3.0.2"
    }
  }
}

provider "docker" {
  host = "unix:///var/run/docker.sock"
}

# Pulls the image
resource "docker_image" "nginx" {
  name = "rmnobarra/nginx:blue"
}

# Create a container
resource "docker_container" "nginx_container" {
  image = docker_image.nginx.image_id
  name  = "nginx_container-${terraform.workspace}"

  ports {
    internal = 80
    external = 8081
  }
}
```

The expected output is two containers with images that have distinct tags, such as:

```bash
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                  NAMES
5ff2648de3b8   b9276eeb87ce   "/docker-entrypoint.…"   4 seconds ago    Up 3 seconds    0.0.0.0:8081->80/tcp   nginx_container-dev
502292894f19   eea310abf40b   "/docker-entrypoint.…"   30 seconds ago   Up 29 seconds   0.0.0.0:8080->80/tcp   nginx_container-prod
```

And a directory with the organization of .tfstate files, separated by workspace, would look like this:

```bash
terraform.tfstate.d
├── dev
│   └── terraform.tfstate
└── prod
    └── terraform.tfstate
```

List available workspaces:


```bash
terraform workspace list
```

Get the current workspace:

```bash
terraform workspace show
```

```hcl
resource "aws_s3_bucket" "example" {
  bucket = "my-app-${terraform.workspace}"
  acl    = "private"
}
```
terraform.workspace variable example:

```hcl
resource "aws_s3_bucket" "example" {
  bucket = "my-app-${terraform.workspace}"
  acl    = "private"
}
```

Deleting a workspace:

```bash
terraform workspace delete <workspace-name>
```

Remove the specified workspace. Attention: this deletes the associated state file but does not remove the provisioned resources.

## Best Practices and Tips for Using Workspaces

* **Consistent naming:** Use clear and consistent names for your workspaces, such as "dev", "staging", "prod", so your team knows exactly which environment they are working in.
* **Avoid dependencies between workspaces:** While workspaces are useful for separating states, they should not be used as substitutes for fully independent environments. Each workspace should be isolated without resource dependencies from other workspaces.
* **Complete environment isolation:** Although workspaces offer state separation, they still share the same infrastructure resources. If different workspaces require completely isolated environments (e.g., separate AWS accounts or distinct VPCs), it may be better to adopt multiple configuration directories or separate Terraform projects.
* **Workspaces do not replace Git branches:** It’s important to remember that workspaces manage only the state, while Terraform configuration code should be managed by version control systems like Git. Use Git branches to control code versions, while workspaces are used to manage environment states.
* **Careful use of variables:** Ensure your Terraform configurations are adaptable to different workspaces by using appropriate variables, such as `terraform.workspace`, to conditionally set values based on the active workspace. This helps avoid provisioning incorrect resources in the wrong environment.

## 7. Variables

Variables in Terraform are essential elements for making configurations more dynamic and reusable. They allow you to easily change infrastructure parameters without directly modifying the configuration code. In this section, we will cover concepts, variable precedence, best practices, related commands, and useful tips for using variables efficiently in Terraform.

## Variable Concept

In Terraform, variables are used to abstract values that may vary between different environments, such as VPC IDs, instance types, regions, and more. This allows the same set of configurations to be used in different contexts without duplicating code.

There are three main types of variables in Terraform:

* **Input Variables:** Define values that can be passed into the Terraform code. They are declared with the `variable` block.
* **Output Variables:** Return values after the code execution, typically used to expose information about the provisioned resources.
* **Local Variables:** Define intermediate values or calculations that are used internally within the code, without being exposed as inputs or outputs.

Example of an input variable declaration:

```hcl
variable "instance_type" {
  description = "The type of instance to use"
  type        = string
  default     = "t2.micro"
}
```

Example of an output variable declaration:

```hcl
output "instance_ip" {
  description = "Endereço IP da instância"
  value       = aws_instance.example.public_ip
}
```

Local variable example:

```hcl
locals {
  environment = "dev"
  instance_count = 3
}
```

## Variable Precedence

Terraform follows a well-defined order of precedence when resolving input variables. This means that a variable’s value can be set in multiple ways, and Terraform will use the value with the highest precedence. The order of precedence is:

1. **Variables defined on the command line** via `-var` or `-var-file`:

```bash
terraform apply -var="instance_type=t2.large"
```

2. **Auto-loaded variable files** (such as `terraform.tfvars` or `terraform.tfvars.json`).

3. **Variable files explicitly passed** via `-var-file`:

```bash
terraform apply -var-file="production.tfvars"
```

Environment variables prefixed with TF_VAR_:

```bash
export TF_VAR_instance_type=t2.medium
```

Default values defined in the variable declaration:

```hcl
variable "example_variable" {
  description = "Uma variável de exemplo com valor padrão"
  type        = string
  default     = "valor_padrao"
}
```

## 8. Style Guide

A style guide aims to standardize how code is written, ensuring it is readable, scalable, and easy to maintain. For Terraform, following certain best practices can help organize and structure files and directories while facilitating team collaboration.

### Code Structure

* **Automatic Formatting:** Run the `terraform fmt` command before each commit to ensure consistent code formatting. This command automatically formats your code according to Terraform’s recommended conventions.
* **Indentation:** Use two spaces for each level of indentation in `.tf` files. This makes the code more readable and helps understand the hierarchy of resources and configurations.
* **Comments:** Use the `#` symbol for comments on one or more lines. Comments help explain complex decisions or provide important context for future code maintainers.

### Naming Resources

Consistency in resource naming is key. Here are some guidelines:

* **Avoid Resource Type in the Name:** Do not include the resource type in the instance name. The type is already described in the resource block, making this redundant.
* **Nouns and Underscores:** Prefer using nouns that describe the resource and separate words with underscores (`_`). Example: `resource "aws_instance" "web_api_instance"`.

### Block and Argument Organization

Maintaining consistency in the order of blocks and arguments improves readability:

* **Simple Arguments First:** When defining resources, place non-block arguments (like `ami` or `instance_type`) at the top.
* **Nested Blocks:** Arguments that include other blocks, such as `lifecycle` or `network_interface`, should be placed after simple arguments, separated by a blank line.

Example code:

```hcl
resource "aws_instance" "example" {
  count         = 1
  ami           = "ami-123456"
  instance_type = "t2.micro"

  network_interface {
    # Nested Block
  }

  lifecycle {
    create_before_destroy = true
  }
}
```

## File Organization

As your Terraform code grows, it’s important to divide resources into logically organized files. Here’s a recommended file structure:

* **main.tf:** Contains all resource and data source blocks.
* **outputs.tf:** Defines all output blocks.
* **variables.tf:** Contains all input variables.
* **providers.tf:** Provider configurations.
* **backend.tf:** Backend configurations.

This structure makes the code easier to navigate and maintain, especially in large teams.

## Moderate Use of Variables

While variables make the code more flexible, overusing them can make maintenance difficult. Define a variable only when there is a real need to configure it for different environments or deployments.

For each variable:

* Always include the type and description.
* Set a default value for optional variables.
* Use the `sensitive = true` parameter for sensitive variables, such as passwords.

Example:

```hcl
variable "instance_count" {
  type        = number
  description = "Number of instances"
  default     = 2
}
```

## Variable Validation
To ensure that provided values meet specific criteria, you can use variable validation. This improves code robustness by preventing incorrect inputs:

Example:

```hcl
variable "instance_count" {
  type        = number
  description = "Number of instances"
  validation {
    condition     = var.instance_count > 0
    error_message = "Number of instances must be greater than zero."
  }
}
```

## Outputs

Always provide a description for each output, as it helps other users understand what is being exposed.

```hcl
output "instance_ip" {
  description = "IP address of instance"
  value       = aws_instance.example.public_ip
}
```

## 9. Loops

Terraform, as a declarative language, provides a clear view of the desired state of infrastructure. However, a feature of declarative languages is the absence of traditional control flow structures like for-loops. Nevertheless, Terraform offers alternatives to efficiently perform loops and conditions using the `count` meta-parameter, the `for_each` expression, and `for` expressions. These tools allow for creating multiple resources, applying repeated blocks, and processing lists or maps dynamically.

## Loops with `count`

The `count` meta-parameter is one of the simplest and oldest ways to create loops in Terraform. It is used to determine how many instances of a resource should be created. When `count` is applied to a resource, Terraform generates multiple copies of that resource, creating an array of resources instead of just one.

```hcl
resource "aws_instance" "example" {
  count = 3

  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```

In this example, three instances will be created based on the value of `count`.

Another Example:

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "3.0.2"
    }
  }
}

provider "docker" {
  host = "unix:///var/run/docker.sock"
}

resource "docker_image" "nginx" {
  name = "rmnobarra/nginx:latest"
}

resource "docker_container" "nginx_container" {
  count = 3
  image = docker_image.nginx.image_id
  name  = "nginx-${count.index}"

  ports {
    internal = 80
    external = 8080 + count.index
  }
}
```

In this example, 3 containers are started, and their names are constructed using the nginx prefix plus the container number according to the value of count.

## Loops with `for_each`

The `for_each` expression is a more advanced way to iterate over lists, sets, or maps, allowing greater control when creating multiple resources or instances of blocks within resources. `for_each` is more flexible than `count` because it works directly with sets and maps, and it allows you to access both the key and value of each item in the loop.

Example using `for_each` for multiple containers:

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "3.0.2"
    }
  }
}

provider "docker" {
  host = "unix:///var/run/docker.sock"
}

resource "docker_image" "nginx" {
  name = "rmnobarra/nginx:green"
}

locals {
  containers = {
    nginx1 = 8080
    nginx2 = 8081
    nginx3 = 8082
  }
}

resource "docker_container" "nginx_container" {
  for_each = local.containers

  image = docker_image.nginx.image_id
  name  = "${each.key}"

  ports {
    internal = 80
    external = each.value
  }
}
```

In this example, three `nginx` containers will be created, each with a distinct name: `nginx1`, `nginx2`, and `nginx3`, using the `for_each` loop to iterate over a set of names.


O `for_each` allows creating multiple containers dynamically, iterating over the `local.containers` map. For each key-value pair in the map, Terraform creates a Docker container with a specific name and port, avoiding code duplication and making the configuration more flexible and scalable.

## Loops with `For Expressions`

For expressions are used to transform lists and maps, allowing you to generate new values based on an existing collection. They are not used to create resources directly but rather to manipulate or generate lists or maps that can be used in other contexts.

Example using a for expression:

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "3.0.2"
    }
  }
}

provider "docker" {
  host = "unix:///var/run/docker.sock"
}

resource "docker_image" "nginx" {
  name = "rmnobarra/nginx:green"
}

locals {
  containers = [for i in range(3) : {
    name = "nginx-${i + 1}"
    port = 8080 + i
  }]
}

resource "docker_container" "nginx_container" {
  count = length(local.containers)

  image = docker_image.nginx.image_id
  name  = local.containers[count.index].name

  ports {
    internal = 80
    external = local.containers[count.index].port
  }
}
```

In this example, the `for` expression creates a map pairing container names with their respective ports. The result is an output that associates `nginx1` with `8081`, `nginx2` with `8082`, and `nginx3` with `8083`.

## When to Use count, for_each, or For Expressions?

* **count:** Use when you need multiple instances of a resource based on a specific number. It is the simplest way to create multiple copies of a resource.
* **for_each:** Use when you need more control over the loop, especially when iterating over sets or maps. This allows creating multiple resources based on more complex collections.
* **For expressions:** These are useful for manipulating lists or maps or applying transformations before passing data to another resource.

## 10 .Modules

Modules in Terraform are one of the key features that make infrastructure as code (IaC) more reusable, organized, and scalable. By dividing configuration into reusable blocks, modules allow you to create, maintain, and share infrastructure configurations efficiently.

## Module Concept

In Terraform, a module is simply any set of `.tf` files in a directory. Whenever you create a configuration in Terraform, you are technically creating a "root module," as the directory containing your configuration is considered a module. However, the real power of modules lies in their ability to be reused in different parts of the code, allowing the creation of reusable and standardized infrastructure blocks.

A module can be as simple as a single resource definition or as complex as a collection of interdependent resources. They enable users to encapsulate parts of the infrastructure, such as networks, instances, or database clusters, and reuse them across different projects or environments.

## Why Use Modules?

* **Code Reusability:** Instead of duplicating code, modules allow you to create it once and reuse it whenever needed.
* **Organization and Maintenance:** Dividing code into modules makes maintenance easier, as each module has a clear and limited purpose.
* **Abstraction:** Modules can abstract infrastructure details, exposing only the necessary parameters while hiding complexity.
* **Scalability:** With well-defined modules, you can scale your infrastructure faster by simply configuring different parameters to reuse the same blocks of code.

## Basic Module Structure

A module consists of a set of `.tf` files within a directory. The module typically contains:

* **main.tf:** The main file that defines resources.
* **variables.tf:** Defines the input variables that the module accepts.
* **outputs.tf:** Defines the output variables that the module returns after execution.

Example of a module structure:

```bash
docker_container_module/
│
├── main.tf
├── variables.tf
└── outputs.tf
```

Example of variables.tf:

```hcl
# variables.tf

variable "image_name" {
  type        = string
  description = "Nome completo da imagem Docker a ser usada"
  default     = "rmnobarra/nginx:latest"
}

variable "container_name" {
  type        = string
  description = "Nome do container Docker"
}

variable "external_port" {
  type        = number
  description = "Porta externa para o container Docker"
}
```

main.tf:

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "3.0.2"
    }
  }
}

resource "docker_image" "nginx" {
  name = var.image_name
}

resource "docker_container" "nginx_container" {
  image = docker_image.nginx.image_id
  name  = var.container_name

  ports {
    internal = 80
    external = var.external_port
  }
}
```

outputs.tf:

```hcl
# outputs.tf

output "container_id" {
  description = "O ID do container Docker criado"
  value       = docker_container.nginx_container.id
}

output "container_name" {
  description = "O nome do container Docker criado"
  value       = docker_container.nginx_container.name
}
```

## Using a Module

To use a module, simply reference it in the main code and pass the required variables. Terraform allows you to use both local and remote modules, such as those available in the Terraform Registry, which offers thousands of ready-to-use modules.

Example of using a previous module:

```hcl
# main.tf

terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "3.0.2"
    }
  }
}

provider "docker" {
  host = "unix:///var/run/docker.sock"
}

module "nginx_container" {
  source = "./docker_container_module" # Caminho para o módulo

  container_name = "my_nginx_container"
  external_port  = 8085
  image_name     = "rmnobarra/nginx:green"
}

output "container_id" {
  value = module.nginx_container.container_id
}

output "container_name" {
  value = module.nginx_container.container_name
}
```

## Conclusion

Terraform has established itself as an essential tool for managing infrastructure as code, offering a declarative and provider-agnostic approach. Its simplicity and flexibility allow development and operations teams to automate and scale their infrastructures efficiently, ensuring consistency and control over environments.

With an active and constantly evolving community, Terraform continues to expand its functionalities and integrations with various providers, making it a robust choice for organizations seeking automation, efficiency, and governance over their cloud infrastructure. Thus, Terraform not only simplifies infrastructure management but also positions itself as a fundamental component in the journey toward adopting modern DevOps practices.

## References

[Official Documentation](https://developer.hashicorp.com/terraform/docs)  
[Terraform: Up and Running: Writing Infrastructure as Code, 3rd Edition](https://amazon.com/Terraform-Running-Writing-Infrastructure-Code/dp/1098116747)


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
