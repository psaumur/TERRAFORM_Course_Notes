# CREATING RESOURCES - PART 3

<ins>Route Table and Internet Gateway</ins>

We have created a custom VPC and a subnet and parameterized the configuration, using variables. The next step to run applications in the virtual private cloud is to configure a route table and an Internet Gateway, for Internet access.

When a VPC is created, other components are created automatically, as well (such as Main route table and Main network ACL).

A ROUTE TABLE is a virtual router within your VPC. It contains routes that determine where the network traffic is directed.

By default, ALL subnets of a VPC are associated with the default ROUTE TABLE of the VPC and traffic that belongs to these subnets will be handled by this route table.

The next step is to create an Internet Gateway, which is a VPC component, that allows communication between your VPC to the Internet.

It basically serves the purpose of routing internet traffic and to perform network address translation (NAT) for instances that have been assigned public internet addresses (IP).

To create an internet gateway, you create an Internet gateway resource:

#Create an Internet Gateway

```jsx
resource "aws_internet_gateway" "my_web_igw" {
vpc_id = aws_vpc.main.id
tags = {
"Name" = "${var.main_vpc_name} IGW"
  }
}
```

---

To create a Default Route Table, you create a Default Route resource:

#Create a Default Route Table

```jsx
resource "aws_default_route_table" "main_vpc_default_rt" {

default_route_table_id = aws_vpc.main.default_route_table_id

route {
cidr_block = "0.0.0.0/0"
gateway_id = aws_internet_gateway.my_web_igw.id
  }
tags = {
"Name" = "my-default-rt"
  }
}
```

Be aware of the IP address for the CIDR Block!

---

The above has now created a public subnet, exposed to the internet, via an Internet Gateway, and has attached it to the Default Route.
