# INTRODUCTION TO TERRAFORM MODULES

As you manage Terraform configurations, you can create increasingly complex configurations and many problems could arise.

Updating the configuration, of a specific section, could become risky because it may cause unintended consequences to other parts of your configuration.

You'll repeat large parts of your configuration for different environments; such as development, staging, or production.

Sharing parts of configurations is not easy.

In a general purpose programming language, if you have the same piece of code copied and pasted in several places, you would normally put that in a code inside of a function and use that everywhere.

There are no user defined functions in Terraform, however there are MODULES.

Terraform modules stick to DRY principles : "Don't Repeat Yourself"

---

Modules help you :

- ORGANIZE CONFIGURATION
- ENCAPSULATE CONFIGURATION
- RE-USE CONFIGURATION
- PROVIDE CONSISTENCY
- ENSURE BEST PRACTICES

MODULES = FUNCTIONS in Programming Languages

A MODULE is a set of configuration files in a single directory

A MODULE can consist only of a single file in a directory.

When you run commands, like `terraform plan` or `terraform apply` directly from such a directory, it is considered the 'Root Module'.

Materials imported from OTHER directories into the 'Root Module' are called 'Child Modules'.

---

There are <ins>TWO</ins> types of MODULES :  Local and Remote

<ins>LOCAL MODULES:</ins>

Loaded from the local file system and are generally created by yourself or other members of your team to organizse and encapsulate your code

<ins>REMOTE MODULES:</ins>

Loaded from a remote source, such as a Terraform Registry and are created and maintained by Hashicorp and it's partners or by third parties.

MODULES are they key to writing reusable, maintainable, and testable Terraform code.

It is GOOD practice to start building everything as a MODULE, create a library of MODULES to share with your team, and start thinking from the beginning of your entire infrastructure, as a collection of reusable modules.

---

<ins>CREATING TERRAFORM MODULES</ins>

A MODULES can consist only of a single file in a directory.

A ROOT MODULE consists of resources defined in the data files in the working directory.

'[main.tf](http://main.tf/)' - main configuration file
'[variables.tf](http://variables.tf/)' - variable definitions for the module.
'[outputs.tf](http://outputs.tf/)' - output definitions for the module

It is recommended for EACH MODULE to declare it's own provider requirements using a required 'providers' block inside a 'terraform' block.

```jsx
terraform {
required_providers {
aws = {
source  = "hashicorp/aws"
version = "~> 4.0"
  }
 }
}
```

This way, Terraform knows that there is a single version of the provider that is compatible with ALL the modules in the configuration.

For convenience, in simple configurations, a CHILD MODULE inherits automatically the default provider configurations from it's PARENT, so you can omit the required provider's block in the CHILD MODULE.

To USE a CHILD MODULE in a ROOT MODULE, you create a module block.

```jsx
module "my_ec2" {
source = "../modules/ec2"
}
```

Note the relative path to the 'modules' folder!

Every time you add/remove a MODULE call from within a configuration, you have to re-run `terraform init`.

Under .terraform in the Project ROOT MODULE folder, a 'modules' and 'providers' folder will be created.

The 'modules.json' file will contain data linking our CHILD module to our ROOT module.

```jsx
{"Modules":[{"Key":"","Source":"","Dir":"."},{"Key":"my_ec2","Source":"../modules/ec2","Dir":"../modules/ec2"}]}
```
