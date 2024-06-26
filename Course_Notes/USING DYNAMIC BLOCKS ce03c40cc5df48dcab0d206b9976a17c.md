# USING DYNAMIC BLOCKS

Another Terraform looping construct, available, is the "dynamic nested block".

In our current FULL Terraform configuration, we have nested blocks of "rules" for our Default Security group.

```jsx
ingress {
from_port = 22
to_port = 22
protocol = "tcp"
cidr_blocks = ["24.207.45.249/32"]
 }

ingress {
from_port = 80
to_port = 80
protocol = "tcp"
cidr_blocks = ["0.0.0.0/0"]
 }

ingress {
from_port = 8080
to_port = 8080
protocol = "tcp"
cidr_blocks = ["0.0.0.0/0"]
 }

egress {
from_port = 0
to_port = 0
protocol = "-1"
cidr_blocks = ["0.0.0.0/0"]
 }
```

Since they all follow the same data structure, these would be perfect for using Dynamic Blocks.

If you have added ALL the commonly used ports, you'd have a long list of entries for each "ingress" and "egress" rule; esp. if you want to customize these rules for multiple instances!

Dynamic Blocks are the answer.

Dynamic Blocks are a loop technique at the ATTRIBUTE level - in this case, "ingress" would be considered an ATTRIBUTE

First, we create a variable list, for the "ingress" ports, in our "[variables.tf](http://variables.tf/)" file

```jsx
variable "ingress_ports" {
description = "List of Ingress Ports"
type = list(number)
default = [ 22,80,110,143,443,993, 8080 ]
 }
```

---

Next, we remove ALL the "ingress" blocks from our "[main.tf](http://main.tf/)" file, except for one (which we will comment out for now)

In a new block, we write:

```jsx
dynamic "ingress" {
for_each = var.ingress_ports
content{
from_port = ingress.value
to_port = ingress.value
protocol = "tcp"
cidr_block = ["0.0.0.0/0"]
 }
}
```

---

'dynamic' defines a new "dynamic block" with the label "ingress".

You can generate nested blocks inside the following block types:
"resource", "data", "provider", and "provisioner"

The 'for_each' call iterates through each number in our variables list (under "default") and assigns it to our nested 'content' variables ('from_port' and 'to_port') via the "ingress.value" call.

The content block is just like our old "ingress" blocks. When the iteration occurs, it's creating a copy into memory, of an "ingress" block, with the data customized by our variable list.

"iterator" is an optional argument that sets the name of a temporary variable that represents the current element of the variable over which you are iterating.

```jsx
dynamic "ingress" {
for_each = var.ingress_ports
iterator = iport
content{
from_port = iport.value
to_port = iport.value
protocol = "tcp"
cidr_blocks = ["0.0.0.0/0"]
 }
}
```

"iport" is the name of our temporary variable.