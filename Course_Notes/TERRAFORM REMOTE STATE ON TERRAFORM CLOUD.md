# TERRAFORM REMOTE STATE ON TERRAFORM CLOUD

We can also host our 'terraform.tfstate' file on Terraform Cloud.

This is a good alternative to Amazon S3 to keep your state file secure and share it with your team.

Create an account

Inside the account, create an Organization

Organization Name : master-terraform53

In our Terraform '[main.tf](http://main.tf/)' file, we need to add a new "block" to our 'required providers' section.

```jsx
terraform {
required_providers {
aws = {
source  = "hashicorp/aws"
version = "~> 4.0"
 }
}
cloud {
organization = "master-terraform53"
workspaces {
name = "DevOps-Production" }
 }
}
```

"organization" refers to the Organization name, we have already added to our 

Terraform Cloud account - it NEEDS to already exist before we can add it to our code.

"workspaces" - creates a workspace name. Does not need to already exist but cannot be reused if it already has a state file in it.

The persistent data stored in the backend, belongs to the workspace and, by default, Terraform starts with a single workspace named "Default"

So, if you've never explicitly used a workspace, then you have only ever worked on the Default.

The Default workspace cannot be deleted and there are backends, Terraform Cloud included, that support multiple named Workspaces.

If you WANT to see what workspace Terraform is using, type:

`terraform workspace show` in a terminal.

(in this instance, it returns 'default')

Now that we've defined the cloud block, we must authenticate to Terraform Cloud to proceed with initialization.

In a terminal, type:

`terraform login` (this will run and open a browser window)

Using our credentials to log into the site, it will ask us to create an API token.

Using the default name ("terraform login"), it creates an API token key, which we can copy/paste into our Terraform login prompt.

When it succeeds, you will see a "Welcome to Terraform Cloud" ascii art message.

Terraform copies your credentials to the home user directory under:
/home/(username)/.terraform.d/credentials.tfrc.json

It's important that you do not lose/erase this key.

Once authenticated, we can now run `terraform init` to initialize our configuration space.

"Terraform Cloud has been successfully initialized!"

We can now run PLAN and APPLY in Terminal, via Terraform Cloud.

---

Running apply in Terraform Cloud. Output will stream here. Pressing Ctrl-C will cancel the remote apply if it's still pending. If the apply started it will stop streaming the logs, but will not stop the apply running remotely.

Preparing the remote apply...

To view this run in a browser, visit:
[https://app.terraform.io/app/master-terraform53/DevOps-Production/runs/run-DhySFpBLtHpNvCHx](https://app.terraform.io/app/master-terraform53/DevOps-Production/runs/run-DhySFpBLtHpNvCHx)

Waiting for the plan to start...

After contacting Terraform Cloud for the state file, it launches our configuration as normal.

If we look at our Organization on Terraform Cloud, we can see it's "Run Status" has changed to "Applied".

("Applied" means the State was saved into the Cloud)

Clicking on the Organization name and we can see the resources that were created, as well as the time taken to launch PLAN and APPLY configurations.

```jsx
new_vpc   hashicorp/aws   aws_vpc   root    Sep 8 2022
Plan & apply duration : less than a minute.
```