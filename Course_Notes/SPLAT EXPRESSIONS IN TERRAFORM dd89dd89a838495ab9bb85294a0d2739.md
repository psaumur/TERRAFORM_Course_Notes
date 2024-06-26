# SPLAT EXPRESSIONS IN TERRAFORM

We create some blocks of code that output the Private IP address of a series of EC2 instances we want to launch.

```jsx
#Display private IP address for first EC2 instance

output "private_address1" {
value=aws_instance.server[0].private_ip
}

#Display private IP address for first EC2 instance

output "private_address2" {
value=aws_instance.server[1].private_ip
}

#Display private IP address for first EC2 instance

output "private_address3" {
value=aws_instance.server[2].private_ip
}
```

But this is inefficient. We can use a SPLAT EXPRESSION that captures all the items in a list that share a common attribute.

```jsx
#Display private IP address for first EC2 instance

output "private_address1" {
value=aws_instance.server[*].private_ip
}
```

Using '*' allows us iterate over all the elements of a given list and returns the information based on the shared attribute.

SPLAT EXPRESSIONS only apply to lists, sets, and tuples and cannot use them with maps or objects.