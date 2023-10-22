## Terraform

### Overview
- Infrastructure as code tool
- Compatible with all of the major cloud providers and projects (it means that it works with both AWS and Kubernetes resources).
- The tool works in a declarative way (same as Kubernetes), which means that I state the wanted infrastructure instead of the steps to reach there.
- Terraform uses providers to manage each cloud provider.
- Terraform uses state files that represent the current state and the tool works to equivalent that with the desired state.

### Usage
- [Terraform installation](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)
- AWS CLI and credentials are required.
- The Terraform's file extension is `.tf`.
- To apply the changes use `terraform init`, observe the changes, then approve.
- While the terraform init, terraform will download the mentioned providers from the terraform registry.

### Syntax

#### Terraform Block
- Contains the settings of terraform including the required providers.
- For each provider, the source attribute defines an optional hostname, a namespace, and the provider type.
- Terraform installs providers from the Terraform Registry by default.
- Example:
  ```
  terraform {
    required_providers {
      aws = {
        source  = "hashicorp/aws"
        version = "~> 4.16"
      }
    }
    required_version = ">= 1.2.0"
  }
  ```

#### Provider Block
- The provider block configures the specified provider.
- Multiple providers together in the same tf file are supported.
- Example:
  ```
  provider "aws" {
    region  = "us-west-2"
  }
  ```
