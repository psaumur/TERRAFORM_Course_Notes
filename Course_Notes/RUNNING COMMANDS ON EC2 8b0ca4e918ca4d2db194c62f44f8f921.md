# RUNNING COMMANDS ON EC2

So far, we've created a VPC, with a subnet, a route table, with an internet gateway (for internet access), defined a security group, and an EC2 instance(based on Amazon Linux 2).

The server is running, however, other than SSH, there is nothing running on it.

We will be running commands or startup scripts on the EC2 Instance at launch-time.

This is ESPECIALLY useful if you want to automate the configuration of your servers.

For example : install / run a Docker container, install and start a web application, add some users/groups, etc.

You *could* SSH into the server and install software THAT way but this is not the best approach because you WANT to automate the configuration process as much as possible; especially if running MANY instances and want/need to maintain it at scale.

Running commands:

You can do this in more than one way.

You can run commands or scripts by giving them a value to an attribute called "user_data"

You can use "cloud-init" which is the industry standard for cloud instance initialization

You can use "Packer" - an open source DevOps tool made by Hashicorp to create images from a single JSON file

OR

You can use provisioners.

---

RUNNING COMMANDS WITH USER_DATA

The "user_data" attribute:

Passes the commands to the cloud provider which runs them on the instance.

Idiomatic and native to the cloud provider.

Viewable in the AWS Console.

We add "user_data" attributes in "[main.tf](http://main.tf/)" under a "resource" block.

```jsx
resource "aws_instance" "my_vm" {
ami = data.aws_ami.latest_amazon_linux2.id
instance_type = "t2.micro"
subnet_id = aws_subnet.web.id
vpc_security_group_ids = [aws_default_security_group.default_sec_group.id]
associate_public_ip_address = true
key_name = aws_key_pair.test_ssh_key.key_name
user_data = <<EOF
#!/bin/bash
sudo yum -y update && sudo yum -y install httpd
sudo systemctl start httpd && sudo systemctl enable httpd
sudo echo "<h1>Deployed via Terraform</h1> > /var/www/html/index.html
EOF
```

---

"<<EOF" means "execute everything below until EOF"

#!/bin/bash - This will run commands using Bash shell

`sudo yum...` - These are our scripted commands and should be
run as 'root' (which is why we are using `sudo...`)

`yum` in this case is due to the "flavor" of Linux the EC2 Instance is using (CentOS / Rocky syntax)

`httpd` is Apache2 and we set it to 'enable' so that the service will be available on reboot of the machine.

'echo...' sends the message to the index file of the website that is served by default to the client (hence the HTML syntax)

You can check the syntax of the commands, before running the configuration, by running them, in the EC2 shell, one at a time to be sure they are correct.

Running the above "user_data" commands, we can look at our EC2 Instance's Public IP address (35.88.131.186) and should see our "Deployed via Terraform" message.

Another way to run "user_data" commands is via scripts.

To do this, you need to create a new file in the root Terraform project folder (in this case: "[entry-script.sh](http://entry-script.sh/)").

Contents of "[entry-script.sh](http://entry-script.sh/)"

```bash
#!/bin/bash
sudo yum -y update && sudo yum -y install httpd
sudo systemctl start httpd && sudo systemctl enable httpd
sudo echo "<h1>Deployed via Terraform</h1>" > /var/www/html/index.html
sudo yum -y install docker
sudo systemctl start docker
sudo usermod -aG docker ec2-user
sudo docker container run -p 8080:80 nginx
```

'-p 8080:80' just exposes port 8080 of nginx docker image to the outside world (via port 80 / http)

Because we are exposing 8080, we need to open it in our Security Group, as well.

---

```jsx
ingress {
from_port = 8080
to_port = 8080
protocol = "tcp"
cidr_blocks = ["0.0.0.0/0"]
cidr_blocks = [var.my_public_ip]
}
```

then change the "user data" entry to:

```jsx
user_data = file("[entry-script.sh](http://entry-script.sh/)")
```

{$file()} : is the call we make when referencing external files.