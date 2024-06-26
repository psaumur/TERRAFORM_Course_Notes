# MANAGING SECRETS IN TERRAFORM

Using Terraform, we can manage the entire infrastructure using code, so the security of the code is crucial.

```jsx
#Configure the AWS Provider
provider "aws" {
region     = "us-west-2"
access_key = "<put your AWS provided Access Key here>"
secret_key = "<put your AWS provided Secret Key here>"
}
```

Looking at the above, we can see that we are storing our IAM 'access_key' and 'secret_key', in plain-text, we anyone with this file can see.

One of the most important steps in security, not just Terraform but any application, is to not store SECRETS in plain-text!

In a Production environment, you should absolutely NEVER store sensitive data in code.

This ALSO includes version control because anyone who has access to the verison control system, has access to that secret.

EVEN IF you store them securely, you still need to secure your Terraform STATE file.

No matter which technique you use to encrypt your secrets, they will still end up in plain-text in the Terraform STATE file.

So:

<ins>TERRAFORM SECURITY RULES</ins>

1. Don't store SECRETS in Plain Text!
2. Store Terraform STATE in a backend that supports encryption.

---

STORING SECRETS USING VARIABLES

We will implement storing secrets inside variables as well as using an RDS Instance resource (Relational Database Service : an isolated DB instance environment in the cloud). A DB Instance can contain multiple user-created databases.

```jsx
resource "aws_db_instance" "default" {
allocated_storage    = 10
engine               = "mysql"
engine_version       = "5.7"
instance_class       = "db.t3.micro"
name                 = "mydb"
username             = "foo"
password             = "foobarbaz"
parameter_group_name = "default.mysql5.7"
skip_final_snapshot  = true
}
```

Before APPLY, we are going to create a variable for the password.

In '[variables.tf](http://variables.tf/)' :

```jsx
variable "db_password" {
type = string
sensitive = true
 }

resource "aws_db_instance" "default" {
allocated_storage    = 10
engine               = "mysql"
engine_version       = "5.7"
instance_class       = "db.t3.micro"
db_name                 = "mydb"
username             = "foo"
password             = var.db_password  <--- refers to variable
parameter_group_name = "default.mysql5.7"
skip_final_snapshot  = true
 }
```

---

Running this Plan, Terraform will prompt you for a password:

```jsx
psaumur@Sanctuary-VirtualBox:~/master_terraform/terraform_security$ terraform apply
var.db_password
Enter a value: mypassword12345

aws_db_instance.default: Creation complete after 4m7s [id=terraform-20220909213809430100000001]
```

---

This worked BUT we can still see the "password" in plain-text within the 'terraform.tfstate' file.

```jsx
"option_group_name": "default:mysql-5-7",
"parameter_group_name": "default.mysql5.7",
"password": "mypassword12345",
"performance_insights_enabled": false,
...
...
...
```

Another way to initialize a Terraform variable is by setting an ENVIRONMENT variable.

At the terminal, you write:

`export TF_VAR_db_password="mypass12345"`

Unfortunately, this is saved in the Bash history in Linux.

BUT if we run the same command with a white-space BEFORE it, it will NOT be saved in Bash history.

`export TF_VAR_db_password="mypass12345"`

THIS will still save the password in plain-text in the STATE file!

---

STORING SECRETS SECURELY USING AWS SECRETS MANAGER

We want to store SECRETS securely and keep plain-text SECRETS out of the code and version control system.

One way to do this, is to use SECRETS storage.

This technique relies on storing your passwords, secret keys, or secrets in general, in a dedicated secret store that is designed specifically for storing sensitive data and controlling access to it.

This way your secrets are stored in a dedicated secret store that enforces encryption and strict access control.

Everything will be defined in the code itself, and there are no extra manual steps or scripts required.

There are a few popular SECRET stores to consider:

Hashicorp Vault (Terraform company), which is open source.
AWS Secrets Manager (AWS managed)
GCP Secret Manager (Google managed)

We are going to see how to configure Terraform to use AWS Secrets Manager. On the AWS Console, search for "AWS Secrets Manager".

Click on "Store a new secret" and select the Secret type.

There are already pre-defined ones but we will select "Other type" and choosing "Plaintext"

```jsx
{
"username":"admin",
"password":"mysecret12345"
}
```

You can add more key/value pairs, if you want. Just use a comma, between them.

We are DONE so we click "Next"

We give the SECRET a name - "db_credentials", leaving the other options at their Default values, and clicking "Next" again.

This page, we can set the Secrets Manager to rotate our SECRETS on a schedule. Useful, in case a Secret was compromised.

Clicking "Next" takes us to a summary of the configuration we just completed. 

Down at the bottom, we can see code samples for different languages on how to retrieve SECRETS in your applications.

We click "Store" to save the SECRET

Now, we need to update our code so that it uses the SECRET in our Secrets Manager.

We add a new data source to our Terraform code.

```jsx
data "aws_secretsmanager_secret_version" "cred"{
secret_id = "db_credentials"
}
```

---

Now , we create a 'locals' value to store the SECRET stored in our dbase to use in our configuration code

```jsx
locals {
db_creds = jsondecode(data.aws_secretsmanager_secret_version.cred.secret_string)
}
```

Now, we can replace our "username" and "password" lines to use our 'locals' variable (the variables used in our SECRET)

```jsx
resource "aws_db_instance" "default" {
allocated_storage    = 10
engine               = "mysql"
engine_version       = "5.7"
instance_class       = "db.t3.micro"
db_name                 = "mydb"
username             = local.db_creds.username
password             = local.db_creds.password
parameter_group_name = "default.mysql5.7"
skip_final_snapshot  = true
 }
```

Now the database password and username was saved securely in the Amazon SECRET store.

KEY TAKEAWAYS:

1. Do not store secrets in plaintext
2. Use environment variables, encrypted files, or a secret store to securely store and retrieve secrets.
3. Use a remote backend that supports encryption such as Amazon S3 or Terraform Cloud.
