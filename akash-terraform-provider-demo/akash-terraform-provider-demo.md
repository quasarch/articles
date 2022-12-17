Hello, in this post I’ll walk you through how you can deploy on the Akash Network through the Akash Terraform Provider.

## What is Terraform?
For those of you who might not be familiar with Terraform, Terraform is an Infrastructure as Code, IaC for short, tool that lets clients define resources in human-readable configuration files (usually .tf files) that they can version, reuse, and share for on-premises or cloud providers. 

Terraform is widely used in today’s enterprise solutions as a way to have a consistent workflow to provision and manage all of their infrastructure throughout its lifecycle and allow for it to evolve collaboratively.

I created this provider to ease and automate the deployment of my infrastructure into Akash and I hope it might clear the path for more enterprise-grade workloads to be deployed on Akash. I’ll make a post about how I integrated it with another tool called Terragrunt to provision large scale infrastructure to the Akash Network.

## Before you start

Before starting make sure you have `akash` installed, as well as `terraform` and `go`.
The code of this provider can be found [here](https://github.com/cloud-j-luna/terraform-provider-akash).

## Demo

To start we’ll first create a folder for our terraform configuration.

```bash
mkdir akash-demo
cd akash-demo
```

Create a file and call it whatever you want, for this demo I’ll simply call it `sdl.yaml` and write the following:
```yaml
version: "2.0"

services:
  website:
    image: nginxdemos/hello
    expose:
      - port: 80
        http_options:
          max_body_size: 104857600
        to:
          - global: true
profiles:
  compute:
    website:
      resources:
        cpu:
          units: 1.0
        memory:
          size: 512Mi
        storage:
          size: 512Mi
  placement:
    akash:
      attributes:
        host: akash
      signedBy:
        anyOf:
          - "akash1365yvmc4s7awdyj3n2sav7xfx76adc6dnmlx63"
      pricing:
        website:
          denom: uakt
          amount: 100
deployment:
  website:
    akash:
      profile: website
      count: 1

```
This is a simple “hello world” nginx application.

Now create a `main.tf` file in the same folder as the SDL file.
First start by defining the dependencies:

```tf
terraform {
  required_providers {
    akash = {
      source = “cloud-j-luna/akash"
      version = “0.0.3”
    }
  }
}
```
At the time you are reading this, the source `cloud-j-luna/akash` might be changed.
This is because I’m still working on the repositories/organization and sorting out where to put them.
If you are not sure which is the latest source check the provider’s repository.

**This is very likely to change to an organisation in the future.**

Now initialize the provider with your settings.
You can either set its parameters on the provider block:

```tf
provider "akash" {
  account_address = “” # Address of the account
  keyring_backend = "os"
  key_name = “” # The key name of your account
  node = "http://akash.c29r3.xyz:80/rpc"
  chain_id = "akashnet-2"
  chain_version = "0.16.4"
}
```

Or you can set the following environment variables and the provider will automatically pick them:

```
| Variable                | Description                                                      |
|-------------------------|------------------------------------------------------------------|
| `AKASH_KEY_NAME`        | Name of your keychain.                                           |
| `AKASH_KEYRING_BACKEND` | Backend of the keyring.                                          |
| `AKASH_ACCOUNT_ADDRESS` | Address of your account.                                         |
| `AKASH_NET`             | Network to use, usually the mainnet.                             |
| `AKASH_VERSION`         | Version of the network.                                          |
| `AKASH_CHAIN_ID`        | Chain id of the network.                                         |
| `AKASH_NODE`            | Akash node to connect to.                                        |
| `AKASH_HOME`            | Absolute path to the Akash's home folder, usually under ~/.akash |
| `AKASH_PATH`            | (Optional) The path to the Akash binary                          |
```

Now for the core of the provider configuration, let’s create the deployment.
Create a resource block and set the `sdl` parameter to the content of the SDL file you created previously.
```tf
resource "akash_deployment" "hello_world" {
  sdl = file("${path.module}/sdl.yaml")
}
```

Now to finalize set the output to print out the service’s URL:

```tf
output "services" {
  value = akash_deployment.hello_world.services
}
```

To run your configuration simply run `terraform init` to initialise the provider and then `terraform apply`.
Review the plan and if you are satisfied with it type yes and wait for the deployment to finish.
Depending on your machine and the network it should take between 20-50 seconds.

Once you see an output similar to the one below, it means your deployment was successfully created.
Notice how the services_uri is empty? This happens sometimes and if you `terraform output` again it should be populated.
If you are not getting the URL for the service you can also check the `lease-status` for your deployment.

```
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

services = tolist([
  {
    "service_name" = "website"
    "service_uri" = ""
  },
])
```

Open a browser and type that URL and check your deployment. You should see a page similar to this:

![Screenshot 2022-07-31 at 22.57.10|690x410](https://global.discourse-cdn.com/standard17/uploads/akash1/original/2X/d/d9a2217f086f8a670facf001c8420feb53e9d3bd.jpeg)

Easy right? Now how can I clean it up when I don’t need it anymore?
With terraform, closing your deployment is as simple as typing `terraform destroy` and approving the changes. You’ll see that after running that command your deployment is closed and the amount is returned from the escrow account to your own.

```
Destroy complete! Resources: 1 destroyed.
```

## Conclusion
This is just the start of your journey of deploying your workloads to the Akash Network through Terraform. You can also check the repository where I store some modules for common applications that run on Akash so that you can further reduce the amount of code you have to write while still maintaining version control over your infrastructure.

I kindly ask you for your feedback on the provider and please leave a star in the [Github repository](https://github.com/cloud-j-luna/terraform-provider-akash) if you found it useful.   I’ll update this post as needed for corrections and improvements. Watch out for future posts and a video demo. :grinning: