# INTRODUCTION TO TERRAFORM REGISTRY

We've established that most Terraform configurations share the same components between them and, by knowing that fact, we are able to break our configurations down into component MODULES that allows us to reuse code.

However, in a lot of ways, it feels like "reinventing the wheel".

Modern software development recommends code reuse through libraries, packages and modules.

Most programming languages allows developers to package and publish these reusable components on a registry (like Pypi for Python, or npm for Node.js, for example).

Terraform is no different. Terraform Registry contains published MODULES for quickly deploying common infrastructure configurations.

---

The Registry comes in <ins>TWO</ins> variants :

<ins>PUBLIC REGISTRY:</ins>
Contains official Terraform providers and community models

<ins>PRIVATE REGISTRY:</ins>
Available as part of the Terraform Cloud and hosts models used internally within an organization.

Anyone can use/publish by provider in the Public Registry.

---

You can also search by provider or module.

For example: 

Searching for 'alb' can bring up an AWS ALB module.

ALB - Application Load Balancer.

This module is used to create a AWS application or network load balancer and it's associated resources.

---

We can also see if something is VERIFIED module if it has a VERIFIED badge next to the module name. Hashicorp verifies modules and are usually higher qualify/proven than UNVERIFIED.

Each MODULE page also contains:

- README's
- Provision Instructions (Configuration code snippet to work with the module)
- Inputs (how to customize the module)
- Outputs
- Dependencies (if there are any)
- Resources
- and so on...

---

<ins>THE VPC MODULE</ins>

Using our prior refactored configuration, we can replace our VPC module with a registry one.

Using modules from the Terraform Registry requires a GitHub account and uses Github to download configurations from the Registry.

```jsx
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
name = var.vpc_name
cidr = var.vpc_cidr_block
azs             = [var.subnet_zone]
public_subnets = [var.web_subnet]
tags = {
	Terraform = "true"
	Environment = "dev"
 }
}
```

---

and set up our "[variables.tf](http://variables.tf/)" and "terraform.tfvars" files to handle our variable defintion needs.

"[variables.tf](http://variables.tf/)"

```jsx
variable "region" {
}
variable "vpc_name"{
}
variable "vpc_cidr_block" {
}
variable "subnet_zone" {
}
variable "web_subnet" {
}

```

"terraform.tfvars"

```jsx
region = "us-west-2"
vpc_name = "DevOps Prod VPC"
vpc_cidr_block = "10.0.0.0/16"
subnet_zone = "us-west-2b"
web_subnet = "10.0.10.0/24"
```

---

<ins>THE SECURITY GROUP MODULE</ins>

Adding SSH access using the AWS "SSH" MODULE

```jsx
module "ssh_server_sg" {
source = "terraform-aws-modules/security-group/aws//modules/ssh"
name        = "ssh"
description = "Security group for SSH ports open within VPC"
vpc_id      = module.vpc.vpc_id
ingress_cidr_blocks = ["0.0.0.0/0"]
}
```

---

<ins>THE EC2 MODULE</ins>

Adding EC2 Instance module

<ins>NOTE:</ins>  We need to define our SSH key pair for our EC2 instance

---

```jsx
resource "aws_key_pair" "test_ssh_key" {
key_name = "testing_ssh_key"
public_key = file(var.public_key)
}
```

---

Declare a local variable for 'public_key' ...

```jsx
variable "public_key" {
}
```

---

then set the path in our "terraform.tfvars" file:

```jsx
public_key = "F:/Terraform/master_terraform/ssh_key/test_rsa.pub"
```

---

```jsx
module "ec2_instance" {
source  = "terraform-aws-modules/ec2-instance/aws"
version = "~> 3.0"
name = "single-instance"
ami                    = var.image_name
instance_type          = var.server_type
key_name               = "testing_ssh_key"
vpc_security_group_ids = [module.ssh_server_sg.security_group_id]
subnet_id              = module.vpc.public_subnets[0]
tags = {
Terraform   = "true"
Environment = "dev"
 }
}
```

---

'image_name' is a local variable defining the AMI we use for our instance launch.

'server_type' refers to the TYPE of server instance launched (in this case "t2.micro"; the free tier one)

These variables are declared in "[variables.tf](http://variables.tf/)" and values defined are in our "terraform.tfvars" file.

module call 'ssh_server' is referring to the module we added for our Security Groups

```jsx
module "ssh_server_sg" {
source = "terraform-aws-modules/security-group/aws//modules/ssh"
}
```

module call "vpc.public_subnets[0]" refers to our VPC module

```jsx
module "vpc" {
source = "terraform-aws-modules/vpc/aws"
}
```

---

These attributes are exposed when calling the MODULES and their attributes are found inside it's documentation under the "inputs" section.

Check OUTPUTS documentation to find what output variables can be assigned to our '[outputs.tf](http://outputs.tf/)' file.

---
