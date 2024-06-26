# TERAFORM STRUCTURE AND PROVIDERS

**What are the steps of the Terraform workflow?**

- WRITE - PLAN - APPLY

WRITE - Write the configuration (infrastructure as code)

PLAN - Preview the changes before applying

APPLY - Provision the infrastructure

---

What is a Terraform Module?

- A container for multiple resources that are used together

What is the Root Module?

- The directory with the configuration files where you will run the 'terraform' command.

What is the common name of the main Terraform configuration file?

- [main.tf](http://main.tf/)

---

**Consider the following configuration snippet:**

```jsx
terraform {
required_providers {
google = {
source = "hashicorp/google"
version = "4.15.0"
  }
 }
}
provider "google" {
# Configuration options
 }
```

What does the above "google" stand for ?

- A local name that was given to the provider to configure

---

Consider the following configuration snippet:

```jsx
terraform {
required_providers {
google = {
source = "hashicorp/google"
version = "4.15.0"
  }
 }
}
provider "google" {
# Configuration options
 }
```

What does "source" represent here?

- A global and unique source address for the provider

Is it allowed to use multiple provider blocks in your Terraform configuration?

- Yes. You can manage resources from different providers in the same configuration file.

---

Consider the following configuration snippet:

```jsx
terraform {
required_providers {
google = {
source = "hashicorp/google"
version = "~>4.15.0"
  }
 }
}
```

What versions of the provider are accepted?

- 4.15.1 and 4.15.10 (point releases of 4.15.0)