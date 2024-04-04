---
layout: post
title: "Deploy infrastructure in Azure Cloud with terraform using github actions"
date: 2024-03-27 10:00:00
categories: [terraform, azure, github actions]
tags: [terraform, azure, github actions, iac]
---

## Introduction

Using IaC (Infrastructure as Code) to deploy resources has become a common practice these days. However, orchestrating an end-to-end journey for users to deploy infrastructure is something that some people within companies may not fully appreciate. For this reason, integrating a Terraform project with GitHub Actions is a very effective strategy.

There are several benefits to using this approach:

True Infrastructure Versioning: There is a single point of execution for Terraform, providing complete traceability of who executed the deployment, when it was executed, and the outcome, whether successful or not.

IaC as Code and Review Process: Merely having Terraform code is not sufficient today. Deploying through a CI/CD (Continuous Integration/Continuous Deployment) pipeline facilitates easy testing, reviewing, securing, and versioning of the code.

Transforming IaC into a Technological Product: By delivering a pipeline that abstracts the Terraform code, and requires users to only provide some data about the infrastructure, the IaC project evolves from a simple project to something much more valuable for the users.

And more, a lot of more...

![Azure with Terraform using Github Actions](https://rmnobarradev.blob.core.windows.net/rmnobarradev/azure-github.png)

## Lets begun

There's five ways to connect Terraform in Azure

- Authenticating to Azure using the Azure CLI
- Authenticating to Azure using Managed Service Identity
- Authenticating to Azure using a Service Principal and a Client Certificate
- Authenticating to Azure using a Service Principal and a Client Secret
- Authenticating to Azure using OpenID Connect

In this example we are using "login With a service principal secret" method. This step can be done running:

`az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/20000000-0000-0000-0000-000000000000"`

Expect output is something like:

```json
{
  "appId": "00000000-0000-0000-0000-000000000000",
  "displayName": "azure-cli-2017-06-05-10-41-15",
  "name": "http://azure-cli-2017-06-05-10-41-15",
  "password": "0000-0000-0000-0000-000000000000",
  "tenant": "00000000-0000-0000-0000-000000000000"
}
```

These values map to the Terraform variables like so:

- appId is the client_id defined above.
- password is the client_secret defined above.
- tenant is the tenant_id defined above.

After creating a service principal in Azure, it is necessary to create a GitHub Secret with its content, in this example, the AZURE_CREDENTIALS secret.

After that, parsing this JSON in a step is sufficient to pass the Terraform environment variables to the pipeline context like:

```hcl
      - name: JSON Parse
        id: parse
        env:
          AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
        run: |
          ARM_CLIENT_ID=$(echo $AZURE_CREDENTIALS | jq -r '.["clientId"]')
          ARM_CLIENT_SECRET=$(echo $AZURE_CREDENTIALS | jq -r '.["clientSecret"]')
          ARM_TENANT_ID=$(echo $AZURE_CREDENTIALS | jq -r '.["tenantId"]')
          ARM_SUBSCRIPTION_ID=$(echo $AZURE_CREDENTIALS | jq -r '.["subscriptionId"]')
          echo ARM_CLIENT_ID=$ARM_CLIENT_ID >> $GITHUB_ENV
          echo ARM_CLIENT_SECRET=$ARM_CLIENT_SECRET >> $GITHUB_ENV
          echo ARM_TENANT_ID=$ARM_TENANT_ID >> $GITHUB_ENV
          echo ARM_SUBSCRIPTION_ID=$ARM_SUBSCRIPTION_ID >> $GITHUB_ENV
```

And a terraform action to execute project

```hcl
      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v3
        with:
         terraform_version: 1.1.7
```

And now, we have a context with all elements to run, init, plan and apply commands

```hcl
      - name: Terraform Init
        run: terraform init

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan

      - name: Terraform Apply
        run: terraform apply --auto-approve
```

The full workflow looks like:

```hcl
name: "Terraform: deploying my infrasctructure"

on: [workflow_dispatch]

jobs:
  terraform-apply:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: infra/azure/my-awesome-terraform-project

    steps:

      - name: Checkout repository
        uses: actions/checkout@v3

      - name: JSON Parse
        id: parse
        env:
          AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
        run: |
          ARM_CLIENT_ID=$(echo $AZURE_CREDENTIALS | jq -r '.["clientId"]')
          ARM_CLIENT_SECRET=$(echo $AZURE_CREDENTIALS | jq -r '.["clientSecret"]')
          ARM_TENANT_ID=$(echo $AZURE_CREDENTIALS | jq -r '.["tenantId"]')
          ARM_SUBSCRIPTION_ID=$(echo $AZURE_CREDENTIALS | jq -r '.["subscriptionId"]')
          echo ARM_CLIENT_ID=$ARM_CLIENT_ID >> $GITHUB_ENV
          echo ARM_CLIENT_SECRET=$ARM_CLIENT_SECRET >> $GITHUB_ENV
          echo ARM_TENANT_ID=$ARM_TENANT_ID >> $GITHUB_ENV
          echo ARM_SUBSCRIPTION_ID=$ARM_SUBSCRIPTION_ID >> $GITHUB_ENV   

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v3
        with:
         terraform_version: 1.1.7
        
      - name: Terraform Init
        run: terraform init

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan

      - name: Terraform Apply
        run: terraform apply --auto-approve
```

*TIP: In the case of a multi-repo Terraform project, it's easy to control which deployment is used by simply changing the working directory. This can be done by using the working-directory parameter like so: - working-directory: infra/azure/my-awesome-terraform-project.*

And finally, place the workflow within a GitHub Actions structure. `.github/workflows/deploy-azure.yaml`

## Considerations

Using this method, it's possible to enhance our IaC delivery. We can add a Checkov Terraform step to increase security, incorporate Terraform linting, or ensure that if a Terraform plan command returns any failures, our pipeline halts before beginning the apply process.

## Conclusion

In just a few steps, we can empower our dear users by enabling them to execute infrastructure tasks themselves, using modules or Terraform projects governed by SRE teams. Furthermore, users can contribute to an inner source process, utilizing Git as the IaC source of truth, for real.

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