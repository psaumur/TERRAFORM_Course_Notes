# TERRAFORM REMOTE STATE AND SECURITY

Terraform is a STATEFUL application - meaning, it keeps track of everything it does in your cloud environment, so that if you need to change something later, Terraform will know what it has already done and make those changes for you.

The 'terraform.tfstate' file keeps track of everything Terraform does (in JSON format and will not exist until you perform at LEAST one `terraform apply`) and prior to ANY operation, Terraform will do a refresh to update the state with the real infrastructure.

Working alone or testing configurations, having a `terraform.tfstate` file and a `.tfstate.backup file`, is great but when working with large, distributed teams, it introduces complications.

Local state is not good for production environments when working in a team.

Concurrency is another issue. It needs a locking mechanism to avoid running into race conditions.

The SOLUTION to these problems:

Store the STATE remotely using a correct backend.

Backends define how operations are executed and where the Terraform STATE is stored.

---

TERRAFORM REMOTE STATE ON AMAZON S3

Amazon S3 is Amazon's managed file store.

We, first, need to create a Bucket on Amazon S3 and configure Terraform to use it for each state.

On AWS (using your IAM account), search for S3 and Create A Bucket.

- Give it a name (like "master_terraform", for example)
- Choose the region to host it in (in my case "us-west-2")
- Object Ownership (left to Default for now)
- Access Control (left to Default for now)
- Block Public Access (left to Default for now)
- Bucket Versioning (Set to Enabled)

Bucket Versioning is important as it will allow us to keep multiple versions of the state file in the bucket. If one gets corrupted, you can recover from the failure.

- Default Encryption (Set to Enabled)

Allows us to use encryption on the state file to protect sensitive data, like plain-text keys / passwords.

Encryption type set to Amazon S3 Key (SSE-S3)

- Advanced Settings (left to Default for now)

Bucket's Name : "masterterraform53"

---

After Bucket creation, we need to configure Terraform to use the remote state from within the S3 Bucket.

Create a file, in the same directory, as our configuration, called '[backend.tf](http://backend.tf/)'

(We copied the example S3 configuration from:
[terraform.io/language/settings/backends/s3](http://terraform.io/language/settings/backends/s3))

```jsx
terraform {
backend "s3" {
bucket = "masterterraform53"
key    = "path/to/my/key"
region = "us-west-2"
 }
}
```

'bucket' is the name of our S3 Bucket (we created above)

'key' is the name of the state file inside the Bucket

'region' is where the Bucket is being held

You will need to add your AWS AIM credentials (access and secret key) to your '[backend.tf](http://backend.tf/)' file, otherwise, launching `terraform init` and `terraform apply` will fail.

---