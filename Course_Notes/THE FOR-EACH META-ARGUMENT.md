# THE FOR-EACH META-ARGUMENT

"for_each" is another meta-argument, like "count", but might work better than "count" in certain conditions.

"for_each" is a great way to create duplicates of resources that can be configured uniquely for each copy.

If you remove an element from a list and use "count" to refer to that list, the elements are ordered, causing all later elements to SHIFT into new index positions.

```jsx
variable "users" {
type = list(string)
default = [ "demo-user","admin1", "john" ]
 }
resource "aws_iam_user" "test" {
name = "${element(var.users, count.index)}"
path = "/system/"
count = "${length(var.users)}"
 }
```

Removing "admin1" and re-running `terraform apply` will cause an error.

What happens is Terraform *tries* to destroy user "admin1" (index 1) then it *tried* to create a resource that was at index2 ("john") at it's new index (index 1) for a resource that already exists (essentially duplicates).

So:

"count" is sensitive to List Order !

The "for_each" argument accepts a 'map' type or a 'set' of strings and creates an instance for EACH item in the map or set.

Sets and Maps DO NOT ALLOW DUPLICATES!

```jsx
variable "users" {
type = list(string)
default = [ "demo-user","john" ]
 }
resource "aws_iam_user" "test" {
for_each = toset(var.users)
path = "/system/"
 } 
```

"toset" is a built-in function that coverts a List to a 'set'.

To access an element in a 'set' you use 'each.key'

```jsx
variable "users" {
type = list(string)
default = [ "demo-user","admin1", "john" ]
}
resource "aws_iam_user" "test" {
for_each = toset(var.users)
name = each.key
path = "/system/"
}
```

Removing any number of elements under "users" / "default" and applying will generate no errors.