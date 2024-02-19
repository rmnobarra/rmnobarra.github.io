---
layout: post
title: "Modifying Terraform Behavior with TF_CLI_ARGS"
date: 2024-02-19 13:00:00
categories: [terraform]
tags: [terraform, devops, iac, sre]
---

## Introduction

Recently, I faced the challenge of needing to add specific functionalities to a Terraform deployment that was tightly integrated (a Docker image encapsulated in Terraform commands within a Pipeline). Time was of the essence, and the alternative of implementing this functionality at the code level would have taken a significant amount of time.

The swift and efficient solution was found in TF_CLI_ARGS.

TF_CLI_ARGS

TF_CLI_ARGS offers a flexible and elegant alternative to directly pass arguments to the Terraform CLI, dynamically and transparently modifying its default behavior. This is particularly useful in CI/CD environments where full control over the entire workflow or the involved applications may not be possible.

How It Works?

Arguments defined in TF_CLI_ARGS are applied immediately after the CLI subcommand, such as plan, apply, and so on. This affects the precedence of command-line arguments over environment variables.

## Practical Examples

Example 1: Automatically Change the Path of .tfvars Files

```bash
export TF_CLI_ARGS="-var-file=/iac/terraform/terraform.tfvars"
```

Passing this argument is equivalent to running terraform apply -var-file=/iac/terraform/terraform.tfvars, but simplifies it to just `terraform apply`.

Example 2: Disable CLI Inputs

To disable any interaction during Terraform execution, we can do:

```bash
export TF_CLI_ARGS_apply="-input=false"
```

This configuration ensures that Terraform apply does not wait for any input, proving to be extremely useful in CI/CD workflows.

Example 3: Modifying Execution Plan Behavior

In a test or development environment, terraform plan can take time due to the Terraform refresh process. We can disable this to speed up execution or work with a well-known state of infrastructure.

```bash
export TF_CLI_ARGS_plan="-refresh=false"
```

This can be very helpful for quick plan executions where the current infrastructure state is not critical or is ephemeral.

Example 4: Setting a Specific Log Level for Troubleshooting

When troubleshooting complex issues in Terraform executions, increasing the verbosity of logs can provide valuable insights. You can set a specific log level for all Terraform commands, facilitating in-depth analysis without modifying individual command executions:

```bash
export TF_CLI_ARGS="-log-level=TRACE"
```

This command configures Terraform to execute all commands with the TRACE log level, the most verbose logging level available, helping to pinpoint the sources of issues more effectively.

## Conclusion

Incorporating TF_CLI_ARGS into your Terraform workflows offers a powerful method to tailor Terraform's behavior to suit specific needs, significantly enhancing automation capabilities, especially in CI/CD pipelines. 

The ability to dynamically adjust execution parameters, disable interactions, change configuration paths, and increase logging detail without altering the core Terraform code or command invocations simplifies complex operations and debugging processes. This flexibility ensures that Terraform can seamlessly integrate into various environments and workflows, making it an invaluable tool for DevOps professionals and Site Reliability Engineers alike. 

By leveraging TF_CLI_ARGS, teams can achieve more efficient, controlled, and customized Terraform operations, leading to smoother deployments and easier management of infrastructure as code.

## References

[Terraform official doc about environment variables.](https://developer.hashicorp.com/terraform/cli/config/environment-variables)

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