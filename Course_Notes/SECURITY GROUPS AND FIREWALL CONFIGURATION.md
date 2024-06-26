# SECURITY GROUPS AND FIREWALL CONFIGURATION

AWS offers us two main layers of security : Security Groups and Network ACLs (Access Control Lists).

SECURITY GROUPS operate at the INSTANCE level (The VM level).

NETWORK ACLs operate at the SUBNET level.

---

SECURITY GROUPS are fundamental of Network Security in AWS.
They control how traffic is permitted into or out of the EC2 instances.

A SECURITY GROUP supports only ALLOW rules (unlike, say, the IP Tables in Linux). ALL internet traffic to a security group is implicitly denied.

SECURITY GROUPS are STATEFUL
This means any changes applied to an Incoming Rule will be automatically applied to the Outgoing Rule, as well.

---

NETWORK ACLs, on the other hand, support both ALLOW and DENY rules, and by default, all traffic is ALLOWED(!).

They are also STATELESS, which means that return traffic must be explicitly allowed by the rules.

---

Both SECURITY GROUPS and NETWORK ACLs can be examined on the VPC Dashboard side menu under SECURITY.

When you create a VPC, a SECURITY GROUP is created as well.

With Terraform, we can configure the default SECURITY GROUP or create a new SECURITY GROUP.

To configure the default SECURITY GROUP of the VPC, we will use a resource called "default_security_group".

---



```jsx
#Create default Security Group
resource "aws_default_security_group" "default_sec_group" {
  vpc_id = aws_vpc.main.id
  ingress {
    from_port = 22 // SSH port
    to_port = 22
    protocol = "tcp"
    # cidr_blocks = ["0.0.0.0/0"]
    cidr_blocks = [var.my_public_ip]
    }

    ingress {
    from_port = 80 // HTTP port
    to_port = 80
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    # cidr_blocks = [var.my_public_ip]
  }
}
```

 

---

To allow traffic for SSH and HTTP, you need two rules for each role.

'ingress' means "to allow access to the cloud" ie: our AWS EC2 instance

'from_port' and 'to_port' are our port-forwarding settings (in this case, port 22). You can also specify a range of ports here.

'protocol' is the accepted connection protocol, set to "tcp"

'cidr_blocks' is the range of IPs that are allowed to SSH INTO the server.

In this instance, this will allow ANY address. If only a few select people should be allowed access, it's a good idea to define the list of accepted IP addresses. 'cidr_blocks' is a list of IP addresses and needs to be wrapped in [] like a list in many computer languages.

If we use a variable here, we can define WHAT IP addresses are allowed SSH access the servers, with this rule.

---


```jsx
#Create default Security Group
resource "aws_default_security_group" "default_sec_group" {
vpc_id = aws_vpc.main.id
ingress {
from_port = 22 // SSH port
to_port = 22
protocol = "tcp"
# cidr_blocks = ["0.0.0.0/0"]
cidr_blocks = [var.my_public_ip]
}
ingress {
from_port = 80 // HTTP port
to_port = 80
protocol = "tcp"
cidr_blocks = ["0.0.0.0/0"]
# cidr_blocks = [var.my_public_ip]
  }
}
```

We use the "egress" keyword when defining rules for OUTWARD traffic, like so:

```jsx
egress {
from_port = 0 // this allows any port
to_port = 0
protocol = "-1" // this allows any protocol
cidr_blocks = ["0.0.0.0/0"]
}
```
