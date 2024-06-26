# CUSTOMIZING TERRAFORM CONFIGURATION WITH VARIABLES  - PART 2

A third way to assign a variable is via the `-var` in the `terraform plan` command.

```jsx
terraform plan -var="web_subnet=10.0.20.0/24"
'terraform plan -var="vpc_cidr_block=192.168.0.0/16" -var="web_subnet="192.168.10.0/24"`
```

TERRAFORM CODE ORGANIZATION

Terraform code can bring a lot of confusion, especially for beginners.

All the data configuration files, in a directory, create a module.

In this case, the root module and Terraform, evaluates all of the configuration files in a module, effectively treating them as a single document.

```
psaumur@Sanctuary-VirtualBox:~/master_terraform/01-aws$
.
├── [main.tf](http://main.tf/)
├── terraform.tfstate
├── terraform.tfstate.backup
├── tfplan
└── [variables.tf](http://variables.tf/)
```

So declarations in those files, such as variables, .tf and the [main.tf](http://main.tf/) are concatenated into a single entity.

If you work with a lot of variables, it makes sense to split them off into a separate file ([variables.tf](http://variables.tf/) or variables.json). Terraform will automatically load variables from a file named "variables".

You can also create a file called "terraform.tfvars" as a variables file, that looks like this:

```jsx
vpc_cidr_block = "10.0.0.0/16"
web_subnet = "10.0.100.0/24"
```

When you use `terraform apply`, and it does NOT find a "terraform.tfvars" file, it will consider the default value for each variable (defined either in "[main.tf](http://main.tf/)" or "[variables.tf](http://variables.tf/)"). 

If no "default" value is defined, it will prompt the user (just like in previous examples).

You can also create variable definition files for different environments such as testing, staging, and production.

By creating a NEW .tfvars file NOT called "terraform.tfvars", you need to specify USING that variable file using the "-var" command line argument.

```jsx
terraform apply -auto-approve -var-file=web-prod.tfvars
terraform apply -auto-approve
aws_vpc.main: Refreshing state... [id=vpc-09575e039cdaf2a45]
aws_subnet.web: Refreshing state... [id=subnet-0181fe63f452b1dbb]
```

![Untitled](CUSTOMIZING%20TERRAFORM%20CONFIGURATION%20WITH%20VARIABLES%2076d08f3eb3f9455aa79c5cb439d3f1ee/Untitled.png)

![Untitled](CUSTOMIZING%20TERRAFORM%20CONFIGURATION%20WITH%20VARIABLES%2076d08f3eb3f9455aa79c5cb439d3f1ee/Untitled%201.png)

![Untitled](CUSTOMIZING%20TERRAFORM%20CONFIGURATION%20WITH%20VARIABLES%2076d08f3eb3f9455aa79c5cb439d3f1ee/Untitled%202.png)

---

```jsx
psaumur@Sanctuary-VirtualBox:~/master_terraform/01-aws$ terraform apply -auto-approve -var-file=web-prod.tfvars
aws_vpc.main: Refreshing state... [id=vpc-00974412c7f40b402]
aws_subnet.web: Refreshing state... [id=subnet-0df3408305a234461]
```

![Untitled](CUSTOMIZING%20TERRAFORM%20CONFIGURATION%20WITH%20VARIABLES%2076d08f3eb3f9455aa79c5cb439d3f1ee/Untitled%203.png)

![Untitled](CUSTOMIZING%20TERRAFORM%20CONFIGURATION%20WITH%20VARIABLES%2076d08f3eb3f9455aa79c5cb439d3f1ee/Untitled%204.png)

---

The last way to define variables in Terraform is to use environment variables.

This is a fallback to the other ways and has the LEAST precedence.

It searches the environment of it's own process for environment variables called TF_var followed by the name of the declared variable.

In the variables declaration file, declare a new variable called "subnet zone"

In the terminal, we can define this new variable using:

`export TF_VAR_subnet_zone="us-west-2b"`  // this looks for availability in area subnet b zone.

You can then make the variable call within our "[main.tf](http://main.tf/)" file, like so.

#Create a Subnet

```jsx
resource "aws_subnet" "web"{
vpc_id = aws_vpc.main.id // which VPC to create the subnet
cidr_block = var.web_subnet // subnet needs to be in the range of the VPC
availability_zone = var.subnet_zone // looks for first availability zone in the us-west-2b region
tags = {
"Name" = "Web Subnet"
  }
}
```

These types of environment variables are not saved across sections if you don't define them in the initialization file in the shell. They are really meant for quick one-offs.

---

VARIABLE PRECEDENCE

There are multiple ways to define variables, so it's important to understand which variables are considered FIRST to LAST.

1. var and -var-file (command line) have HIGHEST precedence.
2. terraform.tfvars file
3. environment variables (TF_VAR_*) have LOWEST precedence.

To insert a value of a variable into a string, you insert the reference to it like this:

`"${var.<variable_name>}"`

Example: 

#Create a VPC

```jsx
resource "aws_vpc" "main" {
cidr_block = var.vpc_cidr_block
tags = {
"Name" = "Production ${var.main_vpc_name}" // this inserts the variable "Main VPC"
  }
}
```
