## Goal

The exercise had two goals:
One was to get more familiar with the Akash deployment process, especially the usage of the Akash terraform provider which is maintained by a good friend of mine ( more about the provider after ).
The second was to setup an online shop to introduce a friend to online retail possibilities.

## Automation

As a software developer and DevOps enthusiast, I never felt comfortable with click-ops or with processes that are not easily reproducible. Deploying in Akash, although possible to automate just using akash-cli, it is not an easy/straightforward process. There are lots of steps involved, and state management of such an endeavor in a bash-style script is not a mundane task.
To tackle that problem, the Akash terraform provider really simplifies the process.
After the deployment in Akash is done, Cloudflare eases the domain registration, SSL/TLS encryption - offloading the burden of managing certificates.
Even that last step can be automated, which means that for each deployment — in which we get a new service URL — Cloudflare DNS will be updated.

### 1st step — Pre-requisites

To be able to run this example you will need the following software on your PATH:
* Terraform
* Provider-services ( akash-cli )

Also, you will need:
* Akash wallet
* Cloudflare account and API token

### 2nd step — create a script that sets required env variables

set-env-variables.sh
```bash 
export AKASH_KEY_NAME=<your-wallet-name>
export AKASH_KEYRING_BACKEND=os
export AKASH_ACCOUNT_ADDRESS="$(provider-services keys show $AKASH_KEY_NAME -a)"
export AKASH_NET="https://raw.githubusercontent.com/ovrclk/net/master/mainnet"
export AKASH_VERSION="$(curl -s "$AKASH_NET/version.txt")"
export AKASH_CHAIN_ID="$(curl -s "$AKASH_NET/chain-id.txt")"
#export AKASH_NODE="https://akash-rpc.polkachu.com:443"
export AKASH_NODE="http://akash.c29r3.xyz/rpc"
export AKASH_HOME="$(realpath ~/.akash)"
export TF_VAR_CLOUD_FLARE_API_TOKEN=<cloud-flare-api-token>
export TF_VAR_CLOUD_ZONE_ID=<cloud-flare-zone-id>
export TF_VAR_WORDPRESS_DB_PASSWORD=<password-for-database>
```
### 3rd step — The good stuff!

Following is the .tf file, which uses 2 providers: the Akash provider and the Cloudflare provider.
One of the highlights is the usage of service_url ( output of Akash provider ) in the Cloudflare provider.

```h
variable "CLOUD_FLARE_API_TOKEN" {
  type = string
  description = "cloudflare api token"
}
variable "CLOUD_CLOUD_ZONE_ID" {
  type = string
  description = "cloudflare api token"
}
variable "WORDPRESS_DB_PASSWORD" {
  type = string
  description = "wordpress database password"
}


terraform {
  required_providers {
    akash = {
      version = "0.0.6"
      source  = "cloud-j-luna/akash"
    }
    cloudflare = {
      source  = "cloudflare/cloudflare"
      version = "~> 3.0"
    }
  }
}
provider "akash" {
  node = "http://akash.c29r3.xyz:80/rpc"
  chain_id = "akashnet-2"
  chain_version = "0.16.4"
}
provider "cloudflare" {
  api_token = CLOUD_FLARE_API_TOKEN
}
variable "zone_id" {
  default = CLOUD_CLOUD_ZONE_ID
}

resource "akash_deployment" "my_deployment" {
  sdl = templatefile("./akash-deployment.yaml",{
    my_sql_root_password = WORDPRESS_DB_PASSWORD
    my_sql_password = WORDPRESS_DB_PASSWORD
    my_sql_database = "wordpress"
    my_sql_user = "wordpress"
  })
}

output "services" {
  value = akash_deployment.my_deployment
}


#cloudflare
resource "cloudflare_record" "www" {
  zone_id = var.zone_id
  name    = "www"
  value   = replace( akash_deployment.my_deployment.services[1].service_uri, "/\\s.*/", "")
  type    = "CNAME"
  proxied = true
}
resource "cloudflare_record" "root" {
  zone_id = var.zone_id
  name    = "@"
  value   = replace(akash_deployment.my_deployment.services[1].service_uri, "/\\s.*/", "")
  type    = "CNAME"
  proxied = true
}
output "deployment_id" {
  value = akash_deployment.my_deployment.id
}

output "deployment_dseq" {
    value = akash_deployment.my_deployment.services[*]
}
```

### 4rd step — Apply

The script assumes the presence of an akash-deployment.yaml file. Providing such a file is not in the scope of this article, but one can be found on Luna’s akash-provider examples on GitHub.
As usual, just run:

```bash
terraform apply --auto-approve
```

## Conclusion

Hope this article helps demonstrate how Akash terraform provider can ease deploying in Akash, while automatically managing Cloudflare DNS to your deployed service.