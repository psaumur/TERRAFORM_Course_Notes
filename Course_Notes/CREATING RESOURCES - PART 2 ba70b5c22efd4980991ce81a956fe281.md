# CREATING RESOURCES - PART 2

## AWS SUBNET

Subnets are created as RESOURCES.

#Create a Subnet

```jsx
resource "aws_subnet" "web"{
vpc_id = aws_vpc.main.id // which VPC to create the subnet
cidr_block = "10.0.10.0/24" // subnet needs to be in the range of the VPC
availability_zone = "us-west-2a" // looks for first availability zone in the us-west-2 region
tags = {
"Name" = "Web Subnet"
  }
}
```

In the above, the subnet resource is called using the
"aws_subnet" call and, in this example, is called "web".

We then refer to the VPC that we wish to subnet:
"aws_vpc.main.id" (referring to our already provisioned VPC)

We then define the CIDR IP block the subnet spans:

"10.0.10.0/24" (whose range needs to fit within the VPC IP range)

"availability_zone" is where we will expand our subnet into. In this case, we use the defined zone "us-west-2" (OREGON) and by adding "a" to the end, it will search WITHIN this zone for the first availability of resources to add our sub-net.

The tag "Name", allows us to name our Subnet and this name is what will appear on our AWS VPS Dashboard info.

Executing `terraform plan`, we get the following:

![Untitled](CREATING%20RESOURCES%20-%20PART%202%20ba70b5c22efd4980991ce81a956fe281/Untitled.png)

![Untitled](CREATING%20RESOURCES%20-%20PART%202%20ba70b5c22efd4980991ce81a956fe281/Untitled%201.png)

---