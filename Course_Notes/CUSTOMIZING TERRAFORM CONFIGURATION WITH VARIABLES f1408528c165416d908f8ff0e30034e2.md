# CUSTOMIZING TERRAFORM CONFIGURATION WITH VARIABLES  - PART 1

Input variables make the Terraform configuration more flexible.

Think of variables as function arguments in a programming language.

These take the form of :

```jsx
variable "vpc_cidr_block" {
}
```

This name will be used to assign a value to a variable from outside and to reference the variables value from within the module.

You can use any word for the variable except SPECIAL KEY WORDS.

To REFERENCE a variable within a Terraform configuration file, we use the `var` keyword, dot, and the variable name.

`var.vpc_cidr_block`

If you run the PLAN, without defining your variables, the user will prompted when you APPLY the configuration.

`var.vpc_cidr_block`

`Enter a value:`

You can also store all your variables in a "[variables.tf](http://variables.tf/)" file.

Variables can hold "default" values, in place, like so:

```jsx
variable "vpc_cidr_block" {
default = "10.0.0.0/24"
 }
variable "web_subnet" {
default = "10.0.10.0/24"
 }
```

You can also use a "description" tag that contains a descriptor for your variables.

```jsx
variable "vpc_cidr_block" {
default = "10.0.0.0/24"
description = "CIDR Block for the VPC"
}
```

"type" determines the data type stored in the variable. In our example, we are using "strings".

```jsx
variable "vpc_cidr_block" {
default = "10.0.0.0/24"
description = "CIDR Block for the VPC"
type = string
}
```

But Terraform supports Booleans and other special "types".

It is recommended to set a description and a type for ALL variables, as well as a Default value; if practical.