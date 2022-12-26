img

Version [0.0.7](https://registry.terraform.io/providers/cloud-j-luna/akash/0.0.7) of the Akash Terraform Provider is out and with it the result of the partnership between Praetor and Quasarch.

To install this version simply import it.
```h
terraform {
  required_providers {
    akash = {
      version = "0.0.7"
      source  = "cloud-j-luna/akash"
    }
  }
}
```

## Overview

The `akash_providers` datasource offers the ability to query the Akash Network for information and metadata regarding the Providers taking part in the network.
Users can specify the minimum uptime required by the provider, this helps filtering out Providers that might not be reliable enough for the type of workload being deployed. The uptime is expressed as a float number between 0 and 100 inclusive.
`all_providers` is a flag to tell Terraform whether it should fetch all the Providers or only the active set.
Perhaps the most important property is the `required_attributes`. With this property users can set the attributes required on the Providers. For example, if a user only wants to get active Providers on region `eu-west`, the following datasource could be declared:

```h
data "akash_providers" "europe_providers" {
  required_attributes = {
    region = "eu-west"
  }
}
```
Usually the following attributes are available on providers:
* `region` - The region of the provider. **As of the date of this post, there is not standard**
* `arch` - Architecture of the provider. E.g amd64, arm64, ...
* `organization`
* `host`
* `tier`
* `cpu` - The CPU of the Provider
* `power` - The source of the power for the Provider. E.g solar, ...

**Note: These values can be set arbitrarily by the Providers**

## Example Usages

### Providers only using solar power

```h
resource "akash_deployment" "my_deployment" {
  sdl = "..."
  provider_filters {
    providers = data.akash_providers.solar_providers.providers.*.address
  }
}
data "akash_providers" "solar_providers" {
  required_attributes = {
    power = "solar"
  }
}
```

### Providers with ARM64

```h
resource "akash_deployment" "my_deployment" {
  sdl = "..."
  provider_filters {
    providers = data.akash_providers.arm64_providers.providers.*.address
  }
}
data "akash_providers" "arm64_providers" {
  required_attributes = {
    arch = "arm64"
  }
}
```

### 100% uptime providers in europe

```h
resource "akash_deployment" "my_deployment" {
  sdl = "..."
  provider_filters {
    providers = data.akash_providers.europe_providers.providers.*.address
  }
}
data "akash_providers" "europe_providers" {
    minimum_uptime = 100
  required_attributes = {
    region = "eu-west"
  }
}
```

## Conclusion

With the data provided by Praetor on providers we can offer the most customizable deployment experience on the Akash Network to date.
You can now be sure your requirements for providers are taken into consideration.

Stay tuned for news in the near future of new partnerships! We have alot of great things to share with you!

\- Quasarch team