# TERRAFORM DATA SOURCES

Looking at the AWS Documentation, we can find AWS Provider / Data Sources

(Search for Data Sources at:
[https://registry.terraform.io/providers/hashicorp/aws/latest/docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs))

What are Data Sources?

They are a kind of API and their goal is to do something and give you back dynamic data.

They are used to fetch dynamic data from cloud providers.

Data Sources are like "queries".

Examples of data sources are a "list of AMIs that changes frequently" or a "list of availability zones".

Suppose we want to launch the same AMI instance, but in ANOTHER region.

In our last example, I launched the AMI in Oregon ("us-west-2")

If I change the region in my Instance to "us-west-1" and subnet to "us-west-1b" and trying to APPLY that plan, I will get an error:

---

```jsx
Error: creating EC2 Instance: InvalidAMIID.NotFound: The image id '[ami-0c2ab3b8efb09f272]' does not exist.
status code: 400, request id: b0aea961-1e9a-45f9-bbf6-188680dedb3b

with aws_instance.my_vm,
on [main.tf](http://main.tf/) line 96, in resource "aws_instance" "my_vm":
96: resource "aws_instance" "my_vm" {
```

---

The reason is that the same AMI ID's are different in each region!

So, we need a way to get the RIGHT AMI dynamically. We cannot hardcode it because it changes with each region and each update.

This is when DATA SOURCES come into play.

---

FILTERING AMI USING DATA SOURCES

A DATA SOURCE is accessed via a special kind of "resource" known as a DATA RESOURCE, declared using a "data" block.

In a Terraform configuration, we create a new resource, using the "data" block.

---

```jsx
data "aws_ami" "latest_amazon_linux2"
```

"aws_ami" is the DATA SOURCE provided by the AWS provider and is used to get the ID of an AMI for use in other resources.

Inside the "data" block, I'll use a few arguments to filter out some AMIs.

// This is a LIST [] of AMI owners to limit the search and
// at least one value must be specified.

owners=["amazon"]

Valid values include :

An account ID
An owner alias, like Amazon, Microsoft, etc.

To FIND the owner of an image on the AWS Management Console, navigate to "Images/AMIs" and select the "Public Images" field from the pull-down menu (left of the search box)

From the listed images, the owner is showing under the "Owner" heads (as is the "Owner alias")

---

Another argument is:

```jsx
most_recent = true
```

---

This is optional. It means, if there are more than one result is returned, it will use the most RECENT one.

We can filter, as well, using a "filter" block.

This will filter the AMI images by name ("amzn2-ami-kernel") - a name taken from the AWS Community AMI list for Amazon Linux instances.

---

```jsx
filter {
name = "name"
values = ["amzn2-ami-kernel-*-x86_64-gp2"]
}
```

---

AMI ID's are listed as:

<type>-<version number>-<cpu type>

You can find more ideas for the "filter" block in the "describe images" command in the offical documentation.

---

We can also "filter" by architecture:

```jsx
filter{
name = "architecture"
values=["x86_64"]
 }
}
```

---

By defining things, in this way, we need to change the ID of the AMI in the Instance resource:

(this part)

```jsx
#Create EC2 Instance
resource "aws_instance" "my_vm" {
ami = "ami-0c2ab3b8efb09f272"
  }
```

TO:

```jsx
#Create EC2 Instance
resource "aws_instance" "my_vm" {
ami = data.aws_ami.latest_amazon_linux2.id
  }
```

---

This points to the variable returned by our:

data "aws_ami" "latest_amazon_linux2" {} call

The reason we *didn't* use a variable, instead of a data source, is because variables are STATIC predefined values, whereas DATA SOURCES are dynamic, fixed values from providers.

---

QUERYING DATA WITH OUTPUTS

Terraform Output Values are used to export structured data about resources.

Output values are like functions' return values

Root module can use outputs to print values at the terminal after running `terraform apply`.

It is a good idea to have all OUTPUTS stored in a separate file called "[outputs.tf](http://outputs.tf/)" under your main Terraform project folder.

Outputs are called using the "output" block.

---

```jsx
output "ec2_public_ip"{
description = "The public IP address of the EC2 Instance"
value = aws_instance.my_vm.public_ip
}
```

When creating an OUTPUT value, it is a good idea to include a description of what is being output. In the case above, the Public IP of our EC2 Instance will be displayed in the console when `terraform apply` is used.

```jsx
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
Outputs: ec2_public_ip = "aws_instance.my_vm.public_ip.id"
```

---

Terraform stores these "output" values, in a state file, which can be queried and are not just displaying in the CLI.

`terraform output` <--- this will show all current outputs.

These can also be filtered by calling the OUTPUT block name.

`terraform output ec2_public_ip` <--- this will return the stored value in this output.

You can also generate JSON files by adding the -json flag to the `terraform output` command.

`terraform output -json`

```jsx
{
"ami_id": {
"sensitive": false,
"type": "string",
"value": "ami-0c2ab3b8efb09f272"
},
"ec2_public_ip": {
"sensitive": false,
"type": "string",
"value": "35.163.103.235"
},
"vpc_ip": {
"sensitive": false,
"type": "string",
"value": "vpc-022f9d3bef2ccc136"
  }
}
```

This output is also called "machine readable output".

You can also redirect this output into a file.

`terraform output -json >outputs.json`

OUTPUT values can be deemed "sensitive" and Terraform will not display them openly in the CLI.

You *normally* use "sensitive" OUTPUTS to share sensitive data like passwords.

To declare an OUTPUT value as "sensitive", you add the "sensitive=true" argument to the output block.

```jsx
output "ami_id" {
description = "ID of AMI"
value = aws_instance.my_vm.ami
sensitive = true
}
```

*** Output by filtering will still show everything !!!
*** Output to JSON will still show everything !!!