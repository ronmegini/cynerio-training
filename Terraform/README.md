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
- `terraform init` used to initialize the terraform project including install the providers, and create the state files inside a hidden `.terraform` directory.
- `terraform fmt` reformat the files to compile with terraform standards.
- `terraform validate` is a linter tool that validate the code.
- `terraform apply` used to apply the changes. First it will print the changes, then you will be asked to approve, and afterthat the resources will applied.

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

#### Resource Block
- Resource block used to define components of the infrastructure.
- A resource might be a physical or virtual component (such as an EC2 instance, or it can be a logical resource such as an application).
- Resource blocks have two strings before the block: the resource type and the resource name.
- The resource type and name are used as a reference for the object. For example, `aws_instance.app_server` will get the VM's ID.
- Resource blocks contain arguments that you use to configure the resource. The arguments are documented in each provider's documentation.
- Example:
  ```
  resource "aws_instance" "app_server" {
    ami           = "ami-830c94e3"
    instance_type = "t2.micro"
    tags = {
      Name = "ExampleAppServerInstance"
    }
  }
  ```
