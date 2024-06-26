# PARAMETERIZING TERRAFORM MODULES

Typically, you organize your configuration into MODULES so that you don't repeat your code, to encapsulate and reuse configurations, as well as provide consistency.

To accomplish this, your module should have a generic purpose and should create a layer of abstraction around the concept in your architecture.

In our CHILD MODULE folder, we created a '[variable.tf](http://variable.tf/)' folder and added the following:

```jsx
variable "ami_id" {
description = "AMI ID to provision"
type = string
default = "ami-0c2ab3b8efb09f272"
}
variable "instance_type" {
description = "Instance type"
type = string
default = "t2.micro"
}
variable "servers" {
description = "Number of instances to create"
type = number
default = 1
}
```

These have replaced our CHILD MODULE's '[main.tf](http://main.tf/)' entries for it's EC2 resource block.

```jsx
#Create an EC2 Instance
resource "aws_instance" "server" {
ami=var.ami_id
instance_type = var.instance_type
count = var.servers
}
```

Note the calls to the variables in the '[variables.tf](http://variables.tf/)' file.

If we want to deviate from our default configuration from the CHILD MODULE, we can customize our ROOT MODULE by adding arguments to the 'module' block

```jsx
module "my_ec2" {
source = "../modules/ec2"
}
```

Currently referencing our CHILD MODULE

Let's say we wanted to override the AMI ID that is used in the instance. We can add the following lines:

```jsx
module "my_ec2" {
source = "../modules/ec2"
ami_id = << add AWS AMI ID here >>
}
```

We can also change the Instance type (currently 't2.micro') and the number of servers created.

```jsx
module "my_ec2" {
source = "../modules/ec2"
ami_id = << add AWS AMI ID here >>
instance_type = "t2.large" <----- Not 'free tier' eligible
servers = 2
}
```

NOTE : Variable names used have to match across MODULES.

If we comment out lines of code, our ROOT module will use the default values specified in our CHILD MODULE (and '[variables.tf](http://variables.tf/)')

---

Having hard-coded values in our configuration is not good practice.

Best practices when adding a '[variables.tf](http://variables.tf/)' and 'terraform.tfvars' file in our ROOT MODULE is to use the SAME variable names, as in our CHILD MODULE.

---