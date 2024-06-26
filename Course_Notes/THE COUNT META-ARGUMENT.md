# THE COUNT META-ARGUMENT

`count` is used to manage similar resources

`count and` for_each` are looping techniques

Using the `count` argument, we can, for example, launch multiple instances of our EC2 servers.

```jsx
resource "aws_instance" "server" {
ami = "ami-08970fb2e5767e3b8"  # Red Hat Linux Ent 8
instance_type = "t2.micro"
count = 3
}
```

---

This will create 3 instances of our server, with distinct infrastructure objects associated with them, and each will be separately created, updated, or destroyed.

---

<ins>CREATING IAM USERS USING COUNT</ins>

IAM = IDENTITY AND ACCESS MANAGEMENT

Fine grained access control to AWS.

With IAM, you can specify WHO can access WHICH services and resources and under WHICH conditions.

To create a single IAM user, with Terraform, create an "aws_iam_user" resource:

```jsx
resource "aws_iam_user" "test" {
name = "x-user"
path = "/system/"
}
```

---

This user will be generated under the IAM dashboard on AWS.

What if we need to create 5, 10, 20 user accounts? Copy and pasting the code would be not optimal due to repetition of code and inefficient.

We can use `count` to loop through creation and use the "index" of count to give a distinct name for each resource.

```jsx
resource "aws_iam_user" "test" {
name = "x-user${count.index}"
path = "/system/"
count = 5
}
```

This will generated accounts for "x-user1","x-user2" and so on.

To set policies for IAM accounts, consult the Terraform AWS documentation.

"count" can be used with module and with EVERY resource type.

We can also define user names by using a variable list.

```jsx
variable "users" {
type = list(string)
default = [ "demo-user","admin1","john" ]
}
```

and use interpolation functions in the "aws_iam_user" block.

These are "element()" and "length()":

"element()" pulls an element from a list object
"length()" pulls the length of a list from a list object

```jsx
resource "aws_iam_user" "test" {
name = "${element(var.users, count.index)}"
path = "/system/"
count = "${length(var.users)}"
}
```

name is pulling the elements from the variable "users" and using the index of "count" to iterate through the list.

count is defined as the "size" (length) of the list of users in variable "users" (in this case, 3)
