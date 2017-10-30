# Terraform Best Practices and Design Patterns, Part 1 

So, you or someone in your company has decided to use [Terraform](https://www.terraform.io) for managing your organization's cloud infrastructure. Good for you. Terraform is the best available tool out there right now for provisioning infrastructure across multiple platforms. If a cloud-based service provider exposes an API, Terraform probably has a wrapper around it. This allows you to manage a ton of different providers with a single syntax.

What Terraform doesn't have is a ton of documentation on what is and isn't good practice. It does have a [recommended practices](https://www.terraform.io/docs/enterprise-beta/guides/recommended-practices/index.html) section, but this mostly focuses on how you should manage team responsibilities in your organization and workspaces, and migrate over into collaborative infrastructure as code.

This series will explore some good (and bad) practices and design patterns for Terraform code by walking through some realistic scenarios. This first post will give some basic best practices that everyone should use on (almost) every Terraform project, before moving on to design patterns.

## Import what you have already

Terraform has this concept of [imports](https://www.terraform.io/docs/import/index.html). It allows you to bring resources that you already have out into the world and add them to your Terraform state. We're going to work from scratch here, but if your company already has a custom VPC on AWS, by all means, get it into Terraform as soon as possible.

## Always use remote state

Terraform has another concept called [remote state](https://www.terraform.io/docs/state/remote.html). Not only should you use it exclusively, but it is what opens up the core of good Terraform design patterns. More on that later.

The obvious advantages of remote state up front are that it lets multiple developers work on the same infrastructure code base without walking over one another. By having a single, canonical location that represents the state of our infrastructure, we don't have to worry about getting out of sync.

This is not the limit of the advantages of remote state, but that alone makes it worth using on every single project. At Hoegg Software, we generally use private AWS s3 buckets for our remote state.

## If at all possible, don't ever edit your state file

Terraform manages the state file for you. If you're using remote state, you'll never even see the thing unless you spit it out to a local file. Leave it alone. Terraform is good at figuring out what it needs to do, and how to get from where things are to where you have declared you want them to be.

## Use AWS profiles

Let's be honest. If you're using Terraform, you're probably also using AWS quite a bit. You don't have to be, but AWS is by far the most popular cloud provider, and Terraform has a ton of support for it.

If you are using AWS, use profiles instead of AWS credentials as variables in your code.

Bad:

```
provider "aws" {
  access_key = "${var.aws_access_key}"
  secret_key = "${var.aws_secret_key}"
  region     = "us-east-1"
}
```

Good:

```
provider "aws" {
  profile = "${var.aws_profile}"
  region  = "us-east-1"
}
```

This protects you from a number of things, including accidently uploading your credentials to your code repo because you forgot/set the `.gitignore` up wrong. If everyone working on the code base has a profile in their AWS credentials file with the name of the `aws_profile` variable, everything will go smoothly.

## Use Chef for configuration management
### AKA: Use Terraform as it was intended

As of right now, the most robust provisioner available for Terraform is [Chef](https://www.terraform.io/docs/provisioners/chef.html). Chef is its own, fully-formed toolset largely beyond the scope of this blog series, but of the available provisioner options right now, Chef is the only one that can easily achieve the [idempotence](https://en.wikipedia.org/wiki/Idempotence) benefits of Terraform. What this means is that you can, if you're using well-designed chef cookbooks, run your builds over and over and get the same outcomes.

The best practices for using Chef with Terraform are the same as the best practices for using Chef in general: wrapper cookbooks around declarative resources, roles over runlists, use the available security tools, and always use the chef-client:service recipe.

All of this is to say that you should not have Terraform handling configuration management with a bunch of Bash/Powershell scripts and null resources. Terraform is a tool for giving you servers and infrastructure that did not previously exist and removing infrastructure you no longer want. It is not a tool for configuration management. Don't try to make it be, or you'll run into headaches.

## Next Time

In part 2, we'll follow these best practices to build our organization a VPC on AWS. We'll use that VPC and the various resources we place in it as a groundwork for illustrating some good code design practices in Terraform.
