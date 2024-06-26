# TERRAFORM PROVISIONERS

What is the difference between passing a bootstrap script using "user_data" and "provisioners" ?

"User_data" is idiomatic because it's native to the cloud provider, to AWS in this case, its the recommended way to customize a new instance.

On the other hand, "provisioners" are SPECIFIC to Terraform.
Terraform will connect itself to the instance using SSH, in most cases, and run commands directly to the instance itself.

"Provisioners" are a last resort because they break the declarative model of Terraform and they also add a considerable amount of complexity and uncertainty to Terraform usage.

- Use "provisioners" with care when there are no other better alternatives.

```jsx
connection {
type = "ssh"
host = self.public_ip
user = "ec2-user"
private_key = file("~/.ssh/test_rsa")
}
provisioner "file" {
source = "./entry-script.sh"
destination = "/home/ec2-user/entry-script.sh"
}
provisioner "remote-exec" {
inline = [
"chmod +x /home/ec2-user/entry-script.sh",
"/home/ec2-user/entry-script.sh",
"exit"
 ]
}
```

---

The above establishes an ssh connection to the instance, using the private key credentials, for user "ec2-user".
Next, it copies our "[entry-script.sh](http://entry-script.sh/)" to the home directory of "ec2-user" and does the following:

- Sets permissions to the script to be executable (by ec2-user)
- Runs the script (sets up httpd and runs an nginx in a docker container to port 8080)
- Exits the SSH connection

---

FAILURE BEHAVIOR

By default, Provisioners that FAIL will also cause Terraform to fail.

We can use an "on_failure" attribute to handle failures of executions like so:

```jsx
on_failure = continue
```

This will cause the execution to continue on any error.