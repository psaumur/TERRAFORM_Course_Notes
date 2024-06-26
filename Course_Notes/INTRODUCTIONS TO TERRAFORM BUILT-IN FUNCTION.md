# INTRODUCTIONS TO TERRAFORM BUILT-IN FUNCTION

Terraform has a number of built-in functions that can be used in expressions to transform and combine values.

(See Terraform reference on Functions)

Types of functions:

- Numeric
- String
- Collection
- Encoding
- Filesystem
- Date and Time
- Hash and Crypto
- IP Network
- Type Conversion

There are no User Defined functions in Terraform.

```jsx
max(10,15,-5)
15

min(2,-18,5)
-18

pow(2,4)
16
```

```jsx
split(",","one,two,three,four")
tolist([
"one",
"two",
"three",
"four",
])

split(":","one:two:three:four")
tolist([
"one",
"two",
"three",
"four",
])
```

```jsx
formatdate("DD-MM-YYYY hh:mm", timestamp())
"06-09-2022 21:25"
```

There are many useful examples of functions in the user documentation.
(See Terraform reference on Functions)

---

<ins>USING TERRAFORM BUILT-IN FUNCTIONS</ins>

The 'lookup' function, takes two arguments, like so:

```jsx
ami=lookup(var.ami, var.region)
```

This refers to the keypairs inside the variable 'ami' contents.

```jsx
variable "ami" {
type = map(string)
default = {
"us-west-1" = "ami-018d291ca9ffc002f"
"us-west-2" = "ami-0c2ab3b8efb09f272"
"us-east-1" = "ami-05fa00d4c63e32376"
"us-east-2" = "ami-0568773882d492fc8"
 }
}

variable "region"{
type=string
default="us-west-2"
 }
```

What this will do is compare the contents of the "ami" variable to the "region" variable. If there's a match, return the "ami" value of that match (in this instance: "ami-0c2ab3b8efb09f272" is returned)

If we want to change the ami code used, we just need to change the variable "region" contents to another region.

```jsx
variable "region"{
type=string
default="us-west-1"
}
```

which would return "ami-018d291ca9ffc002f".

Another in-built function is "file"

```jsx
user_data = file("${path.root}[entry-script.sh](http://entry-script.sh/)")
```

The ${path.root} call refers to the filesystem path of the root module configuration.

The ${path.module} call refers to the filesystem path where the expression is placed.

You can assign functions to locals, as well.

```jsx
locals {
time = formatdate("DD MM YYYY hh:mm", timestamp())
}
```

"formatdate" is a function that formats the data/time from a timestamp() call.

We can output it when we launch our instance via the "[output.tf](http://output.tf/)" file.

```jsx
output "Datetime" {
description = "Current Date and Time"
value=local.time
}
```

'local.time' refers to the locals "time" call.
