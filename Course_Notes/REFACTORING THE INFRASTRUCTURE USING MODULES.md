# REFACTORING THE INFRASTRUCTURE USING MODULES

The best practice when working with MODULES is to group multiple resources into logical units and make an abstraction that is reusable.

In our largest example code base (01-aws), we will systematically dismantle our '[main.tf](http://main.tf/)' components and create modules for each component.

These are :

- The VPC (includes Subnets and Internet Gateway)
- The Security Groups (ingress/egress listing of ports)
- The EC2 Instance

To make variables visible from a CHILD MODULE to a PARENT MODULE, a
CHILD MODULE can use OUTPUTS to expose a SUBSET of resource attributes to the PARENT MODULE.

---

<ins>ACCESSING CHILD MODULE OUTPUT VALUES</ins>

'[outputs.tf](http://outputs.tf/)' entries in CHILD MODULES behave like function return values, accessible globally to the rest of the code.

```jsx
output "main_vpc_id" {
description = "ID of VPC"
value = aws_vpc.main_vpc.id
}
```

We can now access the name of the main VPC in our '[main.tf](http://main.tf/)' PARENT MODULE code by calling <module><module folder name><output variable name>.

```jsx
resource "aws_default_security_group" "default_sec_group" {
vpc_id = module.vpc.main_vpc_id
```

A CHILD Module cannot access the attributes of another CHILD module.

You need to redefine those variables and expose them with it's own '[outputs.tf](http://outputs.tf/)' file to be used in the PARENT Module.

CHILD '[outputs.tf](http://outputs.tf/)' file:

```jsx
output "instance" {
value = aws_instance.my_vm
}
```

PARENT '[outputs.tf](http://outputs.tf/)' file:

```jsx
output "ec2_public_ip" {
description = "The public IP of the EC2 instance"
value = module.server.instance[*].public_ip
 }
output "ami_id" {
description = "ID of AMI"
value= module.server.instance[*].ami
}
```

[*] had to be used due to a "count" call and is wildcard for index number of the instances launched.
