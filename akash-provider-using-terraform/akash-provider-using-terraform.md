# Using the Akash Provider Module

## Introduction

As Infrastructure as Code (IaC) continues to gain traction, Terraform has become a popular choice for managing IT resources. While Terraform offers numerous providers for various platforms, you may occasionally need a custom provider or module for specific services like Akash. In this blog post, we'll guide you through using a Terraform module Akash, enabling you to bootstrap an Akash provider quickly.

## Prerequisites

Before we begin, ensure that you have the following tools installed and configured:

1. Terraform (version 0.13 or higher)
2. Git

### Step 1: Create a Terraform Configuration File

Create a new Terraform configuration file (e.g., provider.tf) and add the following module block:

```hcl
module "provider" {
  source = "<path-to-cloned-repository>"

  # Provider API credentials
  gcp_credentials     = var.gcp_credentials
  gcp_project_id      = var.gcp_project_id
  gcp_region          = var.gcp_region
  gke_service_account = var.gke_service_account
  gke_workload_pool   = var.gke_workload_pool
  k8s_config          = var.k8s_config # define this if you are using your own k8s configuration
}
```

Replace <path-to-cloned-repository> with the actual path to the Akash Provider Terraform module repository you cloned earlier.

### Step 2: Define Required Variables

In the same directory as your provider.tf file, create a variables.tf file to define the required variables:

```hcl
variable "gcp_credentials" {
  type        = string
  description = "Location of service account of GCP"
}

variable "gcp_project_id" {
  type        = string
  description = "GCP Project id"
}

variable "gcp_region" {
  type        = string
  description = "GCP Region"
}

variable "gke_service_account" {
  type        = string
  description = "GKE Service Account Name"
}

variable "gke_workload_pool" {
  type = string
  description = "GKE Workload Pool identifier"
}
# Define additional variables as needed
```

### Step 3: Configure Auto Variables

Create a terraform.auto.tfvars file in the same directory as your provider.tf and variables.tf files. This file will allow you to set values for your variables automatically without needing to provide them as command-line arguments:

```hcl
gcp_credentials     = "<gcp_credentials.json>"
gcp_project_id      = "<gcp_project_id>"
gcp_region          = "<gcp_region>"
gke_service_account = "<gke_service_account>"
gke_workload_pool   = "<gke_workload_pool>"

# Set values for additional variables as needed
```

Replace each variable values with the actual GCP credentials and configuration needed to run the kubernetes cluster.

### Step 4: Initialize and Apply the Terraform Configuration

Initialize your Terraform workspace by running the terraform init command in the directory containing your provider.tf file. This step will download the necessary provider plugins, including the Akash Provider Module:

```sh
terraform init
```

After initialization, review the proposed changes by running terraform plan. If everything looks good, apply the configuration using the terraform apply command:

```sh
terraform apply
```

Conclusion:

Congratulations! You are successfully using the Akash Provider configured through Terraform. This means we can easily start and stop the instance at any moment with little configuration needed.