# PROVISION INFRASTRUCTURE WITH CLOUD-INIT

CLOUD-INIT is the industry standard method for cloud instance initialization.

It's available on most Linux distributions and it's supported by all major cloud providers, private cloud providers, and bare metal installations.

CLOUD-INIT can run "per-instance" or "per-boot" configuration.

It allows you to pass a shell script or a template to your instance that installs or configures the VM to your specifications.

```jsx
data "template_file" "user_data" {
template = "${file("./web-app-template.yaml")}"
}
```

---

In this instance, the user_data script is a .yaml file (another industry standard file structure)

```yaml
---
#cloud-config
#Adding groups to the system

groups:
  - devops: [root,sys]
  - hashicorp

# Adding users to the system. Users are added after groups are added

users:
  - default
  - name: terraform
gecos: terraform
shell: /bin/bash
primary_group: hashicorp
sudo: ALL=(ALL) NOPASSWD:ALL
groups: users, admin
lock_passwd: false
ssh_authorized_keys:
  - ssh-rsa

# Downloading and installing packages

packages:
- httpd
- docker

# Running commands

runcmd:

- sudo systemctl start httpd
- sudo systemctl start docker
- sudo usermod -aG docker ec2-user
- sudo docker run -p 8080:80 nginx
```

Since we are using this, instead of our Bash script from previous chapters, we need to change the variable in our EC2 Instance configuration to point to this .yaml file.

```jsx
user_data = data.template_file.user_data.rendered
resource "aws_instance" "my_vm" {
ami = data.aws_ami.latest_amazon_linux2.id
instance_type = "t2.micro"
subnet_id = aws_subnet.web.id
vpc_security_group_ids = [aws_default_security_group.default_sec_group.id]
associate_public_ip_address = true
key_name = "production_ssh_key"
key_name = aws_key_pair.test_ssh_key.key_name
user_data = data.template_file.user_data.rendered
tags = {
"Name" = "My EC2 Instance - Amazon Linux 2"
 }
}
```