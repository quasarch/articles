## Migrating from traditional cloud to Akash using Terragrunt

In this blog post we’ll be exploring how to integrate the existing Akash Terraform Provider with the Terragrunt tool and migrate an already existing workload run in on traditional cloud to the Akash Network.
This blog post does not in any way imply that you can only achieve this migration through Terragrunt, it’s simply a demonstration of a common setup amount enterprise systems and aims show how they can also start exploring these alternatives

### Important
Make sure you are familiar both with the Akash Terraform Provider and have previously used or read about Terragrunt to take full advantage of the knowledge in this blog post.

**This is not a tutorial on how to use Terragrunt, it is a demonstration on how you can use it to start migrating your large-scale infratructure to the Akash Network**

## Terragrunt

Terragrunt is a thin wrapper that provides extra tools for keeping your configurations DRY (Don’t Repeat Yourself), working with multiple Terraform modules (local and remote), and managing remote state.

When working with Terragrunt there are patterns that greatly improve the scalability and maintainability of you infrastructure code. For a more detailed view on these check the following article https://terragrunt.gruntwork.io/docs/features/keep-your-terragrunt-architecture-dry/. For simplicity consider the following structure:

```
└── live
    ├── terragrunt.hcl
    ├── prod
    │   ├── app
    │   │   └── terragrunt.hcl
    │   └── mysql
    │       └── terragrunt.hcl
    ├── dev
    │   ├── app
    │   │   └── terragrunt.hcl
    │   └── mysql
    │       └── terragrunt.hcl
    └── stage
        ├── app
        │   └── terragrunt.hcl
        └── mysql
            └── terragrunt.hcl
```

## Demo
Let’s base our demo on the same structure shown previously.

```bash
└── live
    ├── terragrunt.hcl    # Variables and cofigurations commonly and inherited by child modules
    ├── prod
    │   ├── app
    │   │   └── terragrunt.hcl
    │   └── mysql
    │       └── terragrunt.hcl
    ├── dev
    │   ├── app
    │   │   └── terragrunt.hcl    # Instantiation of a Terraform module that deploys a specific application
    │   └── mysql
    │       └── terragrunt.hcl    # Instantiation of a Terraform module that contains a MySQL database for the application
    └── stage
        ├── app
        │   └── terragrunt.hcl
        └── mysql
            └── terragrunt.hcl
```

Imagine this is your current IaC deployed on AWS. If you wanted to apply any infrastructure changes to the *dev* environment you would change directory to `./live/dev` and run `terragrunt apply-all` and both modules, `app` and `mysql`, would be applied. **Note that depending on your configurations on child modules, configured values on .`/live/terragrunt.hcl` would also be used if referenced, this keeps code DRY across environments.**

### Migrating the `app` module
Consider you wanted to migrate part of your infrastructure to the Akash Network to save costs. What best component to try this out than the *dev* environment’s `app` module.

Imagine this app is nothing more than a docker container running somewhere that exposes data from a MySQL database through a REST API. That image has only one environment variable which is the connection string for the MySQL database (for simplicity).

We will start by configuring the Terraform Akash Provider with our local setup:

```hcl
# file: ./terragrunt.hcl

generate "provider" {
  path = "provider.tf"
  if_exists = "overwrite_terragrunt"
  contents = <<EOF
provider "akash" {
  account_address = "<your address>"
  keyring_backend = "os"
  key_name = "<your key name>"
  node = "http://akash.c29r3.xyz:80/rpc"
  chain_id = "akashnet-2"
  chain_version = "0.16.4"
}
EOF
}
```
This will generate the `provider.tf` file in the root of all the executed modules so they will all share this configuration. This is important to keep in mind if later you want to have separate accounts for separate environments such as `stage` and `prod` for security or cost management purposes.

This will change based on your setup as it can live alongside an AWS provider configuration as well for other AWS services you might have in your IaC architecture.

Then we’ll use a module that we "supposedly" created that uses the same app’s docker image but has all the SDL configurations abstracted for us and accepts an input variable which is the connection string for the MySQL database for that environment.

```hcl
# file: ./live/dev/app/terragrunt.hcl

include "root" {
    path = find_in_parent_folders()
}

terraform {
    source = "git@github.com:example/modules.git//app?ref=main"
}

dependency “mysql” {
    config_path = “../mysql
}

inputs = {
    connection = dependency.mysql.outputs.connection    # This is oversimplified, usually its a host, port, username and password. The idea is the same.
}
```

Now finally we can run `terragrunt apply-all` and we’ll have our *dev* environment’s app workload running on the Akash Network. The first component in our migration to run in the decentralised cloud.

## Conclusion
With the Akash Terraform Provider it is really easy to start integrating your Web3 workloads with Web2 and start migrating your expensive cloud workloads to the decentralized cloud using the tools you are already familiar with such as Terraform and Terragrunt!

### Important notice
In the version used in this blog post (0.0.4) parallel deployments are not yet supported. If you have several Akash deployments on your modules make sure you do not run them in parallel by setting the `--terragrunt-parallelism=1` flag. This will slow the process of creating your infrastructure, but it is the best way until paralllel deployments are implemented. You can be part of the discussion [here](https://github.com/cloud-j-luna/terraform-provider-akash/issues/8)

## Future
New versions of this blog post will be created with the increase in number of supported features such as the filtering of providers by region (we might want to add an extra layer inside our environments that allows different regions for the workloads).

**It'll also change based on the feedback received.**

:star: Please leave a star in the Github Repository :star:

## Important resources
* [Akash Terraform Provider Github](https://github.com/cloud-j-luna/terraform-provider-akash)
* [Akash Terraform Provider](https://registry.terraform.io/providers/cloud-j-luna/akash/0.0.4)
* [My Twitter](https://twitter.com/luna_4_go)

## Thank you!
Thank you for the Akash Team to make the development of the Akash Terraform Provider possible and for the interest they have been showing. Thank you to the Akash Insiders as well for all the feedback and all the partnerships. Stay tuned for more and better posts.

## What's next?
There are a lot of exciting stuff coming to the Akash Terraform Provider. Stay tuned for some awesome announcements. We are making sure we can grow alongside the Akash Network.
