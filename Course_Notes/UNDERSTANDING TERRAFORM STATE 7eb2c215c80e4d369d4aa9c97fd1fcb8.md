# UNDERSTANDING TERRAFORM STATE

Terraform stores state information about your managed infrastructure.

STATE is used by Terraform to map real-world resources to your configuration, keep track of metadata, and to improve performance.

STATES are stored, by default, in a file called "terraform.tfstate" but can be stored remotely. which works better in a team environment.

The STATE file will only be created after, at least one, `terraform apply`

STATE file will then be updated using `terraform apply` and `terraform destroy`

Even though the STATE file is editable, it is not recommended you directly edit it.

Instead, Terraform provides commands if you want to perform basic modifications to the STATE using the CLI.

How it works :

Terraform compares YOUR configuration file with the STATE file against your existing infrastructure to create plans and make changes to your infrastructure.

SO

The PRIMARY purpose of Terraform STATE is to store bindings between remote objects in the cloud and resources declared in the configuration file.

NOTE:  terraform.tfstate files are backed up, in case the main STATE file is lost or corrupted, to simplify recovery.

STATE files are formatted JSON files.

Being a JSON format, it consists of key value pairs.

"mode" is the type of resource created (managed or data)
"type" is the resource type.
"name" is the local name of the resource.
"provider" is the provider that introduced the resource.
"instances" contains the attributes of the resource.
...

---

THE TERRAFORM STATE COMMAND

We can examine the Terraform STATE, using CLI commands.

`terraform show` will so you all the resources and values in the STATE file.

If you want to find a SPECIFIC resource, you can pass the output of `terraform show` using shell commands.

Example: Show the attributes of the current VPC

`terraform show | grep -A 20 aws_vpc`

- `A 20` means 'show me the first 20 lines that match'

Output:

```jsx
aws_vpc.main:

resource "aws_vpc" "main" {
arn                              = "arn:aws:ec2:us-west-2:193605446602:vpc/vpc-0f6db149f94aea6cc"
assign_generated_ipv6_cidr_block = false
cidr_block                       = "10.0.0.0/16"
default_network_acl_id           = "acl-07c1affe87b496136"
default_route_table_id           = "rtb-02096e57e5b310c41"
default_security_group_id        = "sg-045bdead04a372bec"
dhcp_options_id                  = "dopt-034f25cc358d7f888"
enable_classiclink               = false
enable_classiclink_dns_support   = false
enable_dns_hostnames             = false
enable_dns_support               = true
id                               = "vpc-0f6db149f94aea6cc"
instance_tenancy                 = "default"
ipv6_netmask_length              = 0
main_route_table_id              = "rtb-02096e57e5b310c41"
owner_id                         = "193605446602"
tags                             = {
"Name" = "Production Main VPC"
}
```

---

Example:

Show the resource types and local names

`terraform state list`

```jsx
data.aws_ami.latest_amazon_linux2
aws_default_route_table.main_vpc_default_rt
aws_default_security_group.default_sec_group
aws_instance.my_vm
aws_internet_gateway.my_web_igw
aws_key_pair.test_ssh_key
aws_subnet.web
aws_vpc.main
```

---

Example: The the attributes of the resource type "aws_instance" with the local name "my_vm".

`terraform state show aws_instance.my_vm`

```jsx
resource "aws_instance" "my_vm" {
ami                                  = "ami-0c2ab3b8efb09f272"
arn                                  = "arn:aws:ec2:us-west-2:193605446602:instance/i-05a81b6370eb48f4e"
associate_public_ip_address          = true
availability_zone                    = "us-west-2a"
cpu_core_count                       = 1
cpu_threads_per_core                 = 1
```

etc...

You can also REPLACE resources using the CLI, using the -replace flag to safely recreate resources without first DESTROYING then creating the same resource.

This can be useful in case of system malfunction.

- replace can be used with `terraform plan` or `terraform apply`.

terraform plan -replace="aws_instance.my_vm"

---

```jsx
Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
Outputs: ami_id = <sensitive>
ec2_public_ip = "35.88.131.186"
vpc_ip = "vpc-0f6db149f94aea6cc"
```

---