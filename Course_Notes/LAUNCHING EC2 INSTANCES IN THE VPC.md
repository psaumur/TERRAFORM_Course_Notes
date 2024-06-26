# LAUNCHING EC2 INSTANCES IN THE VPC

---

With our current setup, we've established:

A VPC
A SUBNET
AN INTERNET GATEWAY (to allow internet traffic)
A DEFAULT ROUTE TABLE
SECURITY GROUPS (open ports)

The next logical step is to create and launch an EC2 instance, which is in fact, a VM (virtual machine) that runs in the cloud infrastructure.

To fetch and create an instance, you use a resource called "aws_instance".

On AWS, Search for 'EC2' and 'EC2 Dashboard' then find "Launch Instance".

The first step is to CHOOSE an AMI (Amazon Machine Image).

There are many AMI's to choose from:

- Infrastructure Software
- DevOps
- Business Applications
- Machine Learning
- IoT
- Industries

---

Since we are currently limited to the "Free Tier" selections, we should choose an option from that selection (to avoid unnecessary charges)

"Amazon Linux 2 comes with five years support. It provides Linux kernel 5.10 tuned for optimal performance on Amazon EC2, systemd 219, GCC 7.3, Glibc 2.26, Binutils 2.29.1, and the latest software packages through extras. This AMI is the successor of the Amazon Linux AMI that is now under maintenance only mode and has been removed from this wizard."

Each AMI has a name and an ID (for each version)

Amazon Linux 2 AMI (HVM) - Kernel 5.10, SSD Volume Type ami-0c2ab3b8efb09f272 (64-bit x86)
ami-07c02c38124bd75bd (64-bit Arm)

We NEED its ID in the Terraform configuration file, so copy the ID of the AMI you wish to configure to clipboard so we can paste it into the resource configuration file.

NOTE : AMI ID's change all the time so be aware that the lecture's ID may not match what is currently on AWS.

NOTE : Not all AMI's are available in all AWS regions or that ID's are different in different regions.

---


```jsx
#Create EC2 Instance
resource "aws_instance" "my_vm" {
ami = "ami-0c2ab3b8efb09f272"
}
```

---

We need to add an label called "instance_type" to our resource.

Click 'SELECT' on the AWS AMI page for our resource (that we've copied the ID from) and we will see ALL the TYPES (in a column of options) of instances available to us.

('t2.micro' is the free one.)

Each type shows the number of CPUS, Memory, etc.

---


```jsx
#Create EC2 Instance
resource "aws_instance" "my_vm" {
ami = "ami-0c2ab3b8efb09f272"
instance_type = "t2.micro"
}
```

---

If we want to change our default setup for our instance, we have specify some attributes, like pointing it to use our SUBNET and not our MAIN VPC network, for example.

---


```jsx
#Create EC2 Instance
resource "aws_instance" "my_vm" {
ami = "ami-0c2ab3b8efb09f272"
instance_type = "t2.micro"
subnet_id = aws_subnet.web.id
}
```

---

This forces our EC2 instance to use our SUBNET network IP address and not our MAIN VPC ones.

We can also specify a security group for our EC2, to allow external access to it from the internet.

---


```jsx
#Create EC2 Instance
resource "aws_instance" "my_vm" {
ami = "ami-0c2ab3b8efb09f272"
instance_type = "t2.micro"
subnet_id = aws_subnet.web.id
vpc_security_group_ids = [aws_default_security_group.default_sec_group.id]
}
```

---

Keep in mind SECURITY GROUPS are LISTS since there can be MORE than one SECURITY GROUP !

This uses our already established one that opens Port 22 (SSH) and port 80 (HTTP) to the internet.

To access the Instance from the Internet, we need to set it to associate a publicly available IP for it.

---



```jsx
#Create EC2 Instance
resource "aws_instance" "my_vm" {
ami = "ami-0c2ab3b8efb09f272"
instance_type = "t2.micro"
subnet_id = aws_subnet.web.id
vpc_security_group_ids = [aws_default_security_group.default_sec_group.id]
associate_public_ip_address = true
}
```

---

We can also establish the requirement for an encrypted SSH Key Pair so only people in possession of the key can have access to our EC2 (creating the pair will be covered in later lecture). For now, we will set the variable to require it.

The "key_name" call will generate a key for the instance when it launches.

---


```jsx
#Create EC2 Instance
resource "aws_instance" "my_vm" {
ami = "ami-0c2ab3b8efb09f272"
instance_type = "t2.micro"
subnet_id = aws_subnet.web.id
vpc_security_group_ids = [aws_default_security_group.default_sec_group.id]
associate_public_ip_address = true
key_name = "production_ssh_key"
}
```

---

We close by adding our TAGS to our Instance (such as it's "Name". In this case "My EC2 Instance - Amazon Linux 2")


```jsx
#Create EC2 Instance
resource "aws_instance" "my_vm" {
ami = "ami-0c2ab3b8efb09f272"
instance_type = "t2.micro"
subnet_id = aws_subnet.web.id
vpc_security_group_ids = [aws_default_security_group.default_sec_group.id]
associate_public_ip_address = true
key_name = "production_ssh_key"
tags = {
"Name" = "My EC2 Instance - Amazon Linux 2"
  }
}
```

---

We can preview our PLAN, using `terraform plan` but we still need to establish our SSH keys before we can APPLY it.

---
