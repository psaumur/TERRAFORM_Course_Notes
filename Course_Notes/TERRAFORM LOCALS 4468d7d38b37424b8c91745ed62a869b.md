# TERRAFORM LOCALS

Terraform locals (local values) are named values that you can refer to in your configuration.

They help simplifying the configuration by avoiding repetition and writing more readable code, by using meaningful names, instead of hardcoded values compared to variables.

Compared to variables, locals do no change values during a Terraform run, and are not submitted by users but calculated inside the configuration.

Locals are available only in the current module. They are scoped locally!

Locals can be defined in a file, called "[locals.tf](http://locals.tf/)", in the current directory, in "[variables.tf](http://variables.tf/)" with other variables or even in "[main.tf](http://main.tf/)" (if there are not too many local values.)

Defining locals is the same as variables.

```jsx
locals {
owner = "DevOps Corp Team"
project = "Online Store"
cidr_blocks = ["172.16.10.0/24", "172.16.20.0/24", "172.16.30.0/24"]
common_tags = {
Name = "dev"
Environment = "development"
Version = 1.10
 }
}
```

with key=value pairs as entries.

Local values are "placeholders" that you can reuse throughout your configuration. When you change them in one place, the corresponding values will be updated everywhere.

Whenever you want to use a local value, you use the 'local' keyword, and the name of the key (local.key)

Examples:

```jsx
#Create a VPC
resource "aws_vpc" "main" {
cidr_block = "172.16.0.0/16"
tags = {
"Name" = local.common_tags
 }
}

resource "aws_subnet" "dev_subnets" {
vpc_id = aws_vpc.dev_vpc.id
cidr_block = local.cidr_blocks[0]
availability_zone = "us-west-2b"
tags = {
"Name" = local.common-tags
 }
}

#Create an Internet Gateway Resource

resource "aws_internet_gateway" "dev_igw" {
vpc_id = aws_vpc.dev_vpc.id
tags = {
"Name" = "${local.common-tags["Name"]}"
"Version" = "${local.common-tags["Version"]}"
 }
}
```

---

If you want to see the value of a local variable, use the 'local.key' command in 'terraform console'

```jsx
local.common-tags
{
"Environment" = "development"
"Name" = "dev"
"Version" = 1.1
}

local.common-tags
{
"Environment" = "development"
"Name" = "dev"
"Version" = 1.1
}

local.common-tags
{
"Environment" = "development"
"Name" = "dev"
"Version" = 1.1
}
```