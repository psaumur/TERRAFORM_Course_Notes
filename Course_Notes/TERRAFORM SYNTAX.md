# TERRAFORM SYNTAX

What format can Terraform configuration be written in?

- JSON and the dedicated Terraform language, NACL

---

Consider the following configuration snippet:

```jsx
provider "aws"{
region = "eu-central-1"
 }
resource "aws_vpc" "main_vpc" {
cidr_block = "10.0.0.0/16"
tags = {
"Name" = "Main VPC"
 }
}
```

"provider" and "resource" are called block types
"aws", "aws_vpc", and "main_vpc" are called labels.

---

What type of comment characters are supported by Terraform configuration language?

Answer : #, //, or /* ... /*

What does 'terraform init' do?

- It downloads and installs the providers specified in the configuration file

Where are the provider plugins located?

- In a directory called .terraform that belongs to the current terraform project directory

How can you recover the secret_key used by Terraform to authenticate to AWS?

- It cannot be recovered.

---