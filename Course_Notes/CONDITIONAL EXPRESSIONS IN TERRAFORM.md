# CONDITIONAL EXPRESSIONS IN TERRAFORM

Terraform has the equivalent of an "if" statement using CONDITIONAL EXPRESSIONS and the TERNARY operator.

You can setup your Terraform configuration, like below, but having to comment out code just you run the "right" instance is tedious and prone to error.

```jsx
resource "aws_instance" "test-server" {
ami = "ami-0c2ab3b8efb09f272"
instance_type = "t2.micro"
 }
 
resource "aws_instance" "production-server" {
ami = "ami-0c2ab3b8efb09f272"
instance_type = "t2.micro"
 }
```

---

We can also use "count" statements, which is a shorter step than commenting code:

```jsx
resource "aws_instance" "test-server" {
ami = "ami-0c2ab3b8efb09f272"
instance_type = "t2.micro"
count = 1
 }

resource "aws_instance" "production-server" {
ami = "ami-0c2ab3b8efb09f272"
instance_type = "t2.micro"
count = 0
 }
```

---

And swapping the "count" number to switch which server to launch but there is still a better way.

We create a variable switch called "istest"

```jsx
variable "istest" {
type = bool
}
```

and add/assign a value to it to our terraform.tfvars file:

```jsx
istest = true
```

Then we add our syntax for a CONDITIONAL expression, in "[main.tf](http://main.tf/)" as follows:

```jsx
resource "aws_instance" "test-server" {
ami = "ami-0c2ab3b8efb09f272"
instance_type = "t2.micro"
count = var.istest == true ? 1:0
 }

resource "aws_instance" "production-server" {
ami = "ami-0c2ab3b8efb09f272"
instance_type = "t2.micro"
count = var.istest == false ? 1:0
 }
```

---

How does it evaluate?

It checks if the variable "istest" is "true" then sets the count to "1"; meaning it will launch our test-server instance.
If it is set to "false", it will launch our "production-server", instead.

Note that the "condition expression" can be ANY expression that can resolve to a boolean value. This will usually be an expression that uses equality, comparison, or logical operators.

The "result" values may be of ANY type BUT they must both be of the SAME type.