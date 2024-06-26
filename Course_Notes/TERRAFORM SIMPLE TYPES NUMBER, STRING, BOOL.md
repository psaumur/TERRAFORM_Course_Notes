# TERRAFORM SIMPLE TYPES : NUMBER, STRING, BOOL

VARIABLE TYPES:

Simple:

- number, string, bool, null

Complex:

a) Collection Types: list, map, set

b) Structural Types: tuple, object

Collections:

- Group values of the same type

Structural : 

- Group values of different types

lists are formatted with [...] and type determines what the contents of the list are.

type = list(string) // a list of comma separated quoted strings

Contents of lists are indexed, starting at 0 and called via the list character. [0] returns the value of the first index.

---

maps are formatted with {} and contain "keys" and "values"

#type map

```jsx
variable "amis" {
type = map(string)
default = {
"us-west-1" = "ami-018d291ca9ffc002f",
"us-west-2" = "ami-0c2ab3b8efb09f272",
"us-east-1" = "ami-05fa00d4c63e32376",
"us-east-2" = "ami-0568773882d492fc8"
 }
}
```

To call the "value" stored in a map, you write:

```jsx
ami = var.amis[var.aws_region]
ami = var.amis["${var.aws_region}"] // this the same as above
```

Tuples are similar to lists, except they can contain any data type mix.

```jsx
variable "my_instance" {
type = tuple([string, number, bool])
default = ["t2.micro", 1, true]
}
```

This one is current set to contain a "string", "number", and a "boolean".

Items are stored in a [...] comma-separated structure and can be referred to by their index.

```jsx
instance_type = var.my_instance[0] // value in the first index
```

---

Objects contain a specific set of things of many types.

```jsx
variable "egress dsg" {
type = object({
from_port = number
to_port = number
protocol = string
cidr_block = list(string)
})
default = {
from_port = 0,
to_port = 65365,
protocol = "tcp",
cidr_block = [ "0.0.0.0/0" ],
prefix_list_ids = []
  }
}
```

and the calls from inside "[main.tf](http://main.tf/)"

```jsx
egress {
from_port   = var.egress_dsg["from_port"]
to_port     = var.egress_dsg["to_port"]
protocol    = var.egress_dsg["protocol"]
cidr_blocks = var.egress_dsg["cidr_block"]
prefix_list_ids = []
}
```