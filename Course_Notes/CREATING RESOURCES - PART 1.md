# CREATING RESOURCES - PART 1

Syntax for creating a resource is as follows:

```jsx
resource "<provider>_<resource_type>" "local_name" {
argument1= value1
argument1= value2
...
}
```

Specific syntax for a given resource can be found on the Terraform providers documentation.

Each resource is associated with a single resource type which determines the kind of infrastructure object it manages and what arguments, the other attributes, the resource supports.

Tags (key:value pairs) can be added to resources and are arbitrary.

Can be used for billing information, ownership, automation, access control, etc.

---

<ins>TERRAFORM PLAN AND APPLY</ins>

- Be sure to SAVE your "[main.tf](http://main.tf/)" configuration before running `terraform plan`, otherwise no changes will show.

`terraform plan` 

What is Terraform doing when you execute the above command?

It reads the current state of the existing remote objects to make that each state is up to date.

It compares current configuration to previous configuration then proposes a set of changing actions that should make the remote objects match the current configuration (currently in "[main.tf](http://main.tf/)")

Terraform uses the selected providers to generate the following execution
plan. Resource actions are indicated with the following symbols:

+ create

Terraform will perform the following actions:

![image](https://github.com/psaumur/TERRAFORM_Course_Notes/assets/106411237/33fc7205-8a67-4538-9b5d-eeaf241262de)

To save a PLAN out to a file, you use the :

`terrform plan -out=<<filename>>` command.

Most people using "tfplan" as the output filename.

If we agree with the PLAN, we should execute on the proposed PLAN!

`terraform apply`

when executing APPLY, it authenticates to the cloud provider and prints again the actions that will carry out (ie: What it will do)

If you don’t want to be prompted, you can use the

`terraform apply -auto-approve` command to skip the permission question.

If you output a PLAN to a "tfplan" file, you can execute the plan from the file using:

`terraform apply "tfplan"`

![image](https://github.com/psaumur/TERRAFORM_Course_Notes/assets/106411237/c380100c-e886-456e-a4be-c37ff87e6472)

You can run `terraform plan` again to double-check the current configuration matches the plan that was executed. If it matches, you will receive this message:

`"No changes. Your infrastructure matches the configuration."`

You can delete your provision by using the AWS interface (Delete x) from the Actions pulldown menu on the resource page.

---

<ins>FORMATTING AND VALIDATING CONFIGURATION FILES</ins>

Infrastructure As Code means managing and provisioning your data centers through human and machine readable definition and configuration files. Since you will be working with a LOT of configuration files, keeping them readable and syntatically correct is important!

Terraform CLI includes a sub-command to format the configuration file in an idiomatic or canonical way, and another command to validate the syntax.

`terraform fmt`

This command rewrites your Terraform configuration files so they are readable and consistent.

Naming Conventions:

- Use _ (underscore) instead of -(dash) everywhere.
- Do not repeat resource type in resource name.
/ resource "aws_route_table" "public" {} <--- Correct!
X resource "aws_route_table" "public_route_table" {} <--- Incorrect!

If you want to format the files in another directory, just give that directory as the argument to the `terraform fmt` command.

`terraform fmt 01-aws`

`terraform fmt -diff` will display the difference of formatting changes before rewriting the file.

If you only want to CHECK the changes and not overwrite the file, use "-check"

`terraform fmt -check`

If you want to reformat all the files in subdirectories, use "-recursive"

`terraform fmt -recursive` (by default on the current directory is processed)

This validates the syntax and arguments of the Terraform configuration files in a directory.

`terraform validate`

The command returns errors in the syntax and arguments of the configuration file.

It will not FIX the files for you but will suggest FIXES for you to implement.

---

<ins>DESTROYING INFRASTRUCTURE WITH TERRAFORM</ins>

In addition to building and modifying infrastructure, Terraform can destroy or recreate the resources it manages.

A good practice is that once you no longer need the infrastructure, you may want to destroy it to reduce your security risks and costs.

There are two ways to destroy infrastructure:

1. Running the `terraform destroy` command
2. Remove the resource from configuration file and running `terraform apply`

`terraform destroy`

`psaumur@Sanctuary-VirtualBox:~/master_terraform/01-aws$ terraform destroy
aws_vpc.main: Refreshing state... [id=vpc-0d9aecbe990d07f93]`

Terraform used the selected providers to generate the following execution
plan. Resource actions are indicated with the following symbols:

`- destroy`

Terraform will perform the following actions:

![image](https://github.com/psaumur/TERRAFORM_Course_Notes/assets/106411237/7d8ec9b5-327d-40a9-83df-c82f0d396cd8)

Note the '-' signs, signifying what is being removed.

If you had MORE resources managed by your Terraform project, the command would destroy them all.

If you have more resources and only want to destroy only ONE of them, then you pass the "-target" option to the `terraform destroy` command.

`terraform destroy -target <resource_type.local_name>`

In our [main.tf](http://main.tf/) example, to remove the "aws_vpc" resource, we would use `terraform destroy -target aws_vpc.main` (can add "-auto-approve" to bypass prompt)

<ins>NOTE:</ins>

The "-target" option is not for routine use and is provided only in EXCEPTIONAL situations such as recovering from errors or mistakes, or when Terraform specifically suggests to use it as part of an error message.

Removing resources via the configuration file is the safer and more precise way of managing resources.

---

<ins>REPLACING INFRASTRUCTURE WITH TERRAFORM</ins>

To FORCE a replacement of resources when launching a Terraform PLAN, using the "-replace" flag to `terraform plan` or `terraform apply` and it will safely recreate the resources in your environment.

`terraform apply -replace="aws_instance.server"`

NOTE:

Doing a ‘replace‘ only rebuilds certain resources without doing a full destroy/rebuild.

This can save a LOT of time when working with large instances, running many resources(!)

This is especially useful in the development phase.

Consider the following resource declaration:

#Create a virtual network within the resource group

```jsx
resource "azurerm_virtual_network" "example" {
name = "example-network"
resource_group_name = azurerm_resource_group.example.name
location = azurerm_resource_group.example.location
address_space = ["10.0.0.0/16"]
}
```

What is the unique identifier used to refer to the resource in the local module?

Remember:

`<provider>_<resource_type>" "local_name"`

So : 

`azurem_virtual_network.example`

What command changes the remote infrastructure according to the local definitions found .tf files?

`terraform apply`

`terraform plan` creates an update PLAN and does NOT update the real infrastructure according to "[main.tf](http://main.tf/)".

What Terraform command will validate the configuration files?

`terraform plan`, `terraform init`, or `terraform validate`

What happens when you APPLY a Terraform configuration?

- Terraform makes any infrastructure changes defined in your configuration and updates the state file with any configuration changes it made.
