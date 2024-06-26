# CREATING AN SSH KEY-PAIR FOR EC2

SSH PUBLIC KEY AUTHENTICATION:

An SSH KEY PAIR is a Public and Private key used to authenticate to the EC2 instance.

The Public key is stored on your Instance and the Private key on the client (the person/machine connecting to the instance)

We can manage Key Pairs through the AWS Management Console.

When your Instance is created and boots up for hte first time, the public key of the key pair that you've specified in your '[main.tf](http://main.tf/)' will be placed on your Linux Instance in "authorized_keys", which is a file in a hidden directory called '.ssh/' that resides in the user's home directory.

When you connect to your Linux Instance, using SSH to log in, you must *specify* the Private key that corresponds to the Public key. If you lose your Private key, there is NO way to recover it.

However, if you LOSE your key, there is still a way to connect to your AWS instance -- read the AWS documentation "Connect or your Linux instance if you lose your private key", to learn how.

When generating a key pair, it is TIED to your REGION where you are running your Instance, THEREFORE, it is very important to change the region to the one where you are creating your infrastructure.

In the left-hand navigation pane on the AWS Console Dashboard, under "Network and Security", click "Key Pairs", then "Create key pair" (upper right corner)

Under "Name", use the same name that it's your Terraform resource call (in our case, it's called "production_ssh_key")

We'll currently use the "RSA" type (Default) as it's quite common.

Private Key file format will be .pem (so we can use the OpenSSH client). If we use PuTTY, we'd select .ppk

We can add a tag to the Public key (much like we add Tags in our Terraform file).

When you hit "Create Key", the Private key file will automatically download in the browser.

Copy this .pem into your home directory's .ssh/ folder and change permissions on the file to 400.

`chmod 400 production_ssh_key.pem` 

If you do NOT do this, you will get an error while accessing the key.

NOW we are ready to APPLY our PLAN to create the EC2 Instance.

---

```jsx
ws_vpc.main: Creating...
aws_vpc.main: Creation complete after 1s [id=vpc-0d12a843c92b8fdef]
aws_subnet.web: Creating...
aws_internet_gateway.my_web_igw: Creating...
aws_default_security_group.default_sec_group: Creating...
aws_internet_gateway.my_web_igw: Creation complete after 1s [id=igw-06a94adaefb47f5fe]
aws_subnet.web: Creation complete after 1s [id=subnet-050a8cbdcbd1b95b6]
aws_default_route_table.main_vpc_default_rt: Creating...
aws_default_route_table.main_vpc_default_rt: Creation complete after 1s [id=rtb-0ff715cbdc8e7d0be]
aws_default_security_group.default_sec_group: Creation complete after 3s [id=sg-0553096c3c7080c64]
aws_instance.my_vm: Creating...
aws_instance.my_vm: Still creating... [10s elapsed]
aws_instance.my_vm: Still creating... [20s elapsed]
aws_instance.my_vm: Still creating... [30s elapsed]
aws_instance.my_vm: Still creating... [40s elapsed]
aws_instance.my_vm: Creation complete after 42s [id=i-03c2897d240bcb2d6]

Apply complete! Resources: 6 added, 0 changed, 0 destroyed.
```

---

AUTOMATIC SSH KEY PAIR GENERATION WITH TERRAFORM

We can generate SSH Key Pairs, using Terraform (which is more efficient than using the EC2 interface).

You can USE a Key Pair that already exists or generate a NEW one.

To generate a NEW key-pair, you run :

`ssh-keygen -t rsa -b 2048 -C 'test key' -N '' -f ~/.ssh/test_rsa`

- t = type of encryption
-b = the number of bytes to encrypt the file with
-C = Comment to add to creation
-N = Passphrase to use when generating the key (without this, you will be prompted to enter one)
-f = path / filename to save key as

Now we can configure Terraform to provide a key resource that will be used to control login access to EC2 Instances.

We create a NEW resource type call "aws_key_pair".

#Create SSH Key Pair

```jsx
resource "aws_key_pair" "test_ssh_key"{
key_name = "testing_ssh_key"
public_key = file("/home/psaumur/.ssh/test_rsa.pub")
}
```

---

'test_ssh_key' will be the AWS name for the key
'key_name' is self explanatory
'public_key' is the path/filename of the SSH Public Key (to copy over to the Instance)

file("") is a function that reads the contents of the file and returns them as a string (good for reading a key file).

We also have to change one more thing. In the "aws_instance" resource (where we build our EC2 Instance), we need to change our "key_name" to the name of our new test key.

```jsx
resource "aws_instance" "my_vm" {
ami = "ami-0c2ab3b8efb09f272"
instance_type = "t2.micro"
subnet_id = aws_subnet.web.id
vpc_security_group_ids = [aws_default_security_group.default_sec_group.id]
associate_public_ip_address = true
key_name = aws_key_pair.test_ssh_key.key_name
tags = {
"Name" = "My EC2 Instance - Amazon Linux 2"
 }
}
```

---

Re-running `terraform apply` implements our changes. We know can generate our OWN SSH keys and add them to our Terraform configuration files.

Checking to see if it's working, we can copy the Public IP address from our EC2 Instance (via the AWS EC2 Dashboard) and paste it into our `ssh` command call:

`ssh -i ~/.ssh/test_rsa ec2-user@34.220.150.218`

- NOTE: "ec2-user" is the default username for logging into our EC2 Instance.

It's probably a BETTER idea to use a variable for the Public Key. This way, we don't need to specify the exact file in our "[main.tf](http://main.tf/)" file and change only the "terraform.tfvars" file, when we want to use different keys.

"[main.tf](http://main.tf/)"

```jsx
#Create SSH Key Pair

resource "aws_key_pair" "test_ssh_key"{
key_name = "testing_ssh_key"
public_key = file(var.ssh_public_key)
}
```

---

"terrform.tfvars"

```jsx
ssh_public_key = "/home/psaumur/.ssh/test_rsa.pub"
```

---

"[variables.tf](http://variables.tf/)"

```jsx
variable "ssh_public_key" {
}
```