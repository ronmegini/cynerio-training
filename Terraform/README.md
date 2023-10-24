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
- `terraform init` is used to initialize the terraform project including installing the providers, which are stored inside a hidden `.terraform` directory.
- `terraform fmt` reformat the files to compile with terraform standards.
- `Terraform validates` is a linter tool that validates the code.
- `terraform apply` used to apply the changes. First, it will print the changes, then you will be asked to approve, and after that, the resources will applied.
- After the resources are applied, Terraform creates a `terraform.tfstate` file which stores the state of the objects. This is the ONLY way terraform can know the status of the objects. For that reason, it is forbidden to modify the objects without terraform.
- `terraform show` shows the object state from the state file.
- `terraform destroy` will delete all the objects created by it.

### Syntax

#### Terraform Block
- Contains the settings of terraform including the required providers.
- For each provider, the source attribute defines an optional hostname, a namespace, and the provider type.
- Terraform installs providers from the [Terraform Registry](https://registry.terraform.io) by default.
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

#### Varibles
- Usually defined in the `variables.tf` file
- Static/default values defined inside of a `variable "<name>"` {} block. Example:
  ```
  variable "instance_name" {
    description = "Value of the Name tag for the EC2 instance"
    type        = string
    default     = "ExampleAppServerInstance"
  }
  ```
- Variables access is possible via var.<var_name>. Example:
  ```
  tags = {
    Name = var.instance_name
  }
  ```
- Variables also can be set by `-var` flag in the apply command. Example:
  ```
  -var "instance_name=YetAnotherName"
  ```
- Output values to present useful information to the Terraform user during the apply process or by the `terraform output` command. It is defined in the `outputs.tf` file, for example:
  ```
  output "instance_id" {
    description = "ID of the EC2 instance"
    value       = aws_instance.app_server.id
  }
  ```
#### Data Source
- Allows data to be fetched or computed for use elsewhere in Terraform configuration.
- The data object is imported from the provider.
- Example:
  ```
  data "aws_availability_zones" "available" {}
  ```
  The variables can be retrieved by: `data.aws_availability_zones.available.names`
