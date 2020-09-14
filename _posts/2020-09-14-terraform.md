---
title: 'Terraform for Dummies'
date: 2020-09-14 00:00:00
excerpt: Preparing for (and passing) the Terraform Associate Certification.
categories: [cloud]
tags: [cloud, aws, gcp, azure]
featured_image: '/images/feature/1.jpg'
scroll_image: '/images/terraform/tf_associate.png'
comments: true
share: true
---

# [Abstract](#abstract)
We are building a platform at work that needs to support Terraform infrastructure-as-code. I decided to get ramped up on the technology so that I wouldn't be talking out of my ass about it. As part of getting ramped up, I found out that Hashicorp offers a certification for Terraform, so I decided to pursue it. 

This article is basically a rehash of the [Terraform Study Guide](https://learn.hashicorp.com/tutorials/terraform/associate-study). The notes here are basically bullet points that are written from my perspective (lots of experience with IaC and cloud in general). There's also an egregious amount of copy pasta.

**These notes are in no way comprehensive.** They're just what I felt I needed to sit the certification exam and feel comfortable talking about Terraform at work. Here are some things that I don't really get into: Terraform Cloud, modules, module registries, resource dependencies, expressions, operators, complex types, functions, and dynamic blocks. I would check these out in the documentation if you don't know what they are. 

Lastly, after passing the certification, I would recommend spending a little extra time studying everything about `variables`. This article is light on the subject and I wish I was a little more knowledgable about them. 

---
## [CLI Tips and Tricks](#cli-tips-and-tricks)

Enable tab completion. If you use either `bash` or `zsh`, you can enable tab completion for Terraform commands. To enable autocomplete, run the following commands.
```shell
$ terraform -install-autocomplete
$ exec bash
```

I also set up  `$ alias tf="terraform"` (similar to `k="kubectl"` for all my Kuberhomies out there.)

---
## [Providers](#providers)
Terraform includes the [meta-argument](https://www.terraform.io/docs/configuration/providers.html#alias-multiple-provider-configurations) `alias` for configuring a `provider` multiple ways for different resources. You can select which provider configuration to use on a per-resource or per-module basis. The primary reason for this is to support multiple regions for a cloud platform; other examples include targeting multiple Docker hosts, multiple Consul hosts, etc.

---
## [Terraform State](#terraform-state)

[State locking](https://www.terraform.io/docs/state/locking.html) is default behavior if your backend supports it. You can manually override the lock, but it's dangerous to do so. 

*When Terraform is used to manage larger systems, teams should use multiple separate Terraform configurations that correspond with suitable architectural boundaries within the system so that different components can be managed separately and, if appropriate, by distinct teams. Workspaces alone are not a suitable tool for system decomposition, because each subsystem should have its own separate configuration and backend, and will thus have its own distinct set of workspaces.*

You can use backends that support remote state to create an [operating model for infrastructure consumption across teams](https://www.terraform.io/docs/state/remote.html#delegation-and-teamwork):  
*For example, a core infrastructure team can handle building the core machines, networking, etc. and can expose some information to other teams to run their own infrastructure. As a more specific example with AWS: you can expose things such as VPC IDs, subnets, NAT instance IDs, etc. through remote state and have other Terraform states consume that.*

*Terraform state can contain sensitive data, depending on the resources in use and your definition of "sensitive." The state contains resource IDs and all resource attributes. For resources such as databases, this may contain initial passwords.*  
*If you manage any sensitive data with Terraform (like database passwords, user passwords, or private keys), treat the state itself as sensitive data.*  

---
## [Terraform Settings](#terraform-settings)
Beyond tinkering with the CLI, you can alter Terraform settings within a configuration file, like so:
```terraform
terraform {
   # ...
}
```
Within a `terraform` block, only constant values can be used; arguments may not refer to named objects such as resources, input variables, etc, and may not use any of the Terraform language built-in functions.

You can use this block to configure things like:
  - [Backend configuration](https://www.terraform.io/docs/configuration/backend.html)
  - Specifying a version of Terraform
  - Specifying [Provider Requirements](https://www.terraform.io/docs/configuration/provider-requirements.html)
  - Opt-in to [experimental language features](https://www.terraform.io/docs/configuration/terraform.html#experimental-language-features)
  - Passing [metadata](https://www.terraform.io/docs/internals/provider-meta.html) to Providers

---
## [Provisioners Are a Last Resort](#provisioners-are-a-last-resort)
Direct quote from [the docs](https://www.terraform.io/docs/provisioners/index.html):
*Provisioners can be used to model specific actions on the local machine or on a remote machine in order to prepare servers or other infrastructure objects for service.*

*Terraform includes the concept of provisioners as a measure of pragmatism, knowing that there will always be certain behaviors that can't be directly represented in Terraform's declarative model.*

*However, they also add a considerable amount of complexity and uncertainty to Terraform usage. Firstly, Terraform cannot model the actions of provisioners as part of a plan because they can in principle take any action. Secondly, successful use of provisioners requires coordinating many more details than Terraform usage usually requires: direct network access to your servers, issuing Terraform credentials to log in, making sure that all of the necessary external software is installed, etc.*

You can also set up [destroy-time provisioners](https://www.terraform.io/docs/provisioners/#destroy-time-provisioners) using a conditional `when` statement:
```terraform
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Destroy-time provisioner'"
  }
}
```

---
## [Master the Workflow](#master-the-workflow)

The [Core Terraform Workflow](https://www.terraform.io/guides/core-workflow.html) has three steps:
  1. Write
  2. Plan
  3. Apply

As teams and the infrastructure grows, so does the number of sensitive input variables (e.g. API Keys, SSL Cert Pairs) required to run a plan. It's best practice to have a robust Continuous Integration pipeline to make the iterative process easier and more secure. Not only does this make writing IaC better, but it helps to review pull requests because a CI pipeline is able to automatically include the output of `tf plan` along with the PR. 

---
## [Learn More Subcommands](#learn-more-subcommands)
You are able to [import](https://www.terraform.io/docs/import/index.html) existing resources into the Terraform state. Currently, Terraform is not able to generate a configuration for the resource when importing, so you will need to author the resource configuration yourself. Be sure that each resource you import is mapped to only one Terraform resource address.

The `terraform fmt` command is used to [rewrite Terraform configuration files](https://www.terraform.io/docs/commands/fmt.html) to a canonical format and style. Tack on the `-diff` flag to display all formatting changes:  
```shell
$ tf fmt -diff
```

The `terraform validate` command [validates the configuration files](https://www.terraform.io/docs/commands/validate.html) in a directory, referring only to the configuration and not accessing any remote services such as remote state, provider APIs, etc.  
```shell
$ tf validate
```
Validate is safe to run automatically. Running `terraform plan` also performs a validation, but pulls in the contextual information about a run, like the target workspace, input variables, etc. 

The `terraform show` command is used to provide human-readable output from a state or plan file. This can be used to inspect a plan to ensure that the planned operations are expected, or to inspect the current state as Terraform sees it. You can also generate machine-readable output by using the `-json` flag. 

The `terraform refresh` command is used to reconcile the state Terraform knows about (via its state file) with the real-world infrastructure. This can be used to detect any drift from the last-known state, and to update the state file.

### [Tainting](#tainting)
Direct quote from [Get Started - AWS](https://learn.hashicorp.com/tutorials/terraform/aws-provision?in=terraform/aws-get-started) on HashiCorp Learn:

*If a resource successfully creates but fails during provisioning, Terraform will error and mark the resource as "tainted". A resource that is tainted has been physically created, but can't be considered safe to use since provisioning failed.*

*When you generate your next execution plan, Terraform will not attempt to restart provisioning on the same resource because it isn't guaranteed to be safe. Instead, Terraform will remove any tainted resources and create new resources, attempting to provision them again after creation.*

*Terraform also does not automatically roll back and destroy the resource during the apply when the failure happens, because that would go against the execution plan: the execution plan would've said a resource will be created, but does not say it will ever be deleted. If you create an execution plan with a tainted resource, however, the plan will clearly state that the resource will be destroyed because it is tainted.*

You can also manually taint resources using `$ tf taint resource.id`

---
## [Use and Create Modules](#use-and-create-modules)
Use the `version` attribute in the `module` block to specifcy verions:
```terraform
module "consul" {
  source  = "hashicorp/consul/aws"
  version = "0.0.5"

  servers = 3
}
```

---
## [Lifecycle Blocks](#lifecycle-blocks)
Within a resource block, you can detail [special lifecycle behavior](https://www.terraform.io/docs/configuration/resources.html#lifecycle-lifecycle-customizations) for resources using the nested `lifecycle` block. The following meta-arguments are available to *all* resources, regardless of type.

  - `create_before_destroy`: Creates a new resource before deleting the existing one when set to `true`. Used when updates cannot be applied in-place.
  - `prevent_destroy`: When set to `true`, will cause Terraform to reject with an error any plan that would destroy the infrastructure object associated with the resource, as long as the argument remains present in the configuration.
  - `ignore_changes`: Accepts a list of attribute names as parameters. This attributes are ignored when applying future configuration updates. This is useful for the (rare) circumstances where a system outside of Terraform changes the value of an attribute and it should not be rolled back by Terraform. 

---
## [Local-only Resources](#local-only-resources)
*While most resource types correspond to an infrastructure object type that is managed via a remote network API, there are certain specialized resource types that operate only within Terraform itself, calculating some results and saving those results in the state for future use.*

*For example, [local-only resource types](https://www.terraform.io/docs/configuration/resources.html#local-only-resources) exist for generating private keys, issuing self-signed TLS certificates, and even generating random ids. While these resource types often have a more marginal purpose than those managing "real" infrastructure objects, they can be useful as glue to help connect together other resources.*

*The behavior of local-only resources is the same as all other resources, but their result data exists only within the Terraform state. "Destroying" such a resource means only to remove it from the state, discarding its data.*

---
## [Operation Timeouts](#operation-timeouts)
Certain resource types have nested `timeout` [block arguments](https://www.terraform.io/docs/configuration/resources.html#operation-timeouts) available to them. These blocks allow you to customize how long certain operations are allowed to take before being considered to have failed. 

---
## [Data Sources](#data-sources)
Data sources allow Terraform to access information that is outside of the configuration file itself. The data is retrieved using a `data` block. 
```terraform
data "aws_ami" "example" {
  most_recent = true

  owners = ["self"]
  tags = {
    Name   = "app-server"
    Tested = "true"
  }
}
```


---
# [Resources](#resources) 
  - [Terraform Study Guide](https://learn.hashicorp.com/tutorials/terraform/associate-study)
  - [Terraform Tutorials](https://learn.hashicorp.com/terraform)
  - [Terraform Documentation](https://www.terraform.io/docs/index.html)
  - [Terraform GitHub](https://github.com/hashicorp/terraform)  