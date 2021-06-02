# 05 - Terraform Modules

## Expected Outcome

In this challenge, you will create a module to contain a scalable virtual machine deployment, then create an environment where you will call the module.

## How to

### Create Folder Structure

From the Cloud Shell, change directory into a folder specific to this challenge. If you created the scaffolding in Challenge 00, then then you can use the command `cd ~/AzureWorkChallenges/challenge05/`.

In order to organize your code, create the following folder structure with `main.tf` files.

```sh
├── environments
│   └── dev
│       └── main.tf
└── modules
    └── my_virtual_machine
        └── main.tf
```

### Create the Module

Inside the `my_virtual_machine` module folder copy over the terraform configuration from challenge 04.

### Create the Environment

Change your working directory to the `environments/dev` folder.

Update main.tf to declare your module, it could look similar to this:

```hcl
variable "username" {}
variable "password" {}

module "myawesomewindowsvm" {
  source = "../../modules/my_virtual_machine"
  name   = "awesomeapp"
}
```

> Notice the relative module sourcing.

### Terraform Init

Run `terraform init`.

```sh
Initializing modules...
- myawesomewindowsvm in ../../modules/my_virtual_machine

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/azurerm versions matching "2.61.0"...
- Installing hashicorp/azurerm v2.61.0...
- Installed hashicorp/azurerm v2.61.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

We have a problem! We didn't set required variables for our module.

Update the `main.tf` file:

```hcl
module "myawesomewindowsvm" {
  source = "../../modules/my_virtual_machine"
  name   = "awesomeapp"
  vm_size  = "Standard_A2_v2"
  username = "${var.username}"
  password = "${var.password}"
}
```

Run `terraform init` again, this time there should not be any errors.

## Terraform Plan

Run `terraform plan` and you should see your linux VM built from your module.

```sh
  + module.myawesomewindowsvm.azurerm_resource_group.module
      id:                                 <computed>
      location:                           "centralus"
      name:                               "awesomeapp-rg"

...

Plan: 6 to add, 0 to change, 0 to destroy.
```

## Add Another Module

Add another `module` block describing another set of Virtual Machines:

```hcl
module "differentwindowsvm" {
  source = "../../modules/my_virtual_machine"
  name   = "differentapp"
  vm_size  = "Standard_A2_v2"
  username = "${var.username}"
  password = "${var.password}"
}
```

## Scale a single module

Set the count of your first module to 2 and rerun a plan.

```hcl
...
    vmcount = 2
...
```

Run a plan and observer that your first module can scale independently of the second one.

## Terraform Plan

Since we added another module call, we must run `terraform init` again before running `terraform plan`.

We should see twice as much infrastructure in our plan.

```sh
  + module.myawesomewindowsvm.azurerm_resource_group.module
      id:                                 <computed>
      location:                           "centralus"
      name:                               "awesomeapp-rg"

...

  + module.differentlinuxvm.azurerm_resource_group.module
      id:                                 <computed>
      location:                           "centralus"
      name:                               "differentapp-rg"

...

Plan: 12 to add, 0 to change, 0 to destroy.

```

## More Variables

In your `environments/dev/main.tf` file we can see some duplication and secrets we do not want to store in configuration.

Add two variables to your environment `main.tf` file for username and password.

Create a new file and name it `terraform.tfvars` that will contain our secrets and automatically loaded when we run a `plan`.

```hcl
username = "testadmin"
password = "Password1234!"
```

## Terraform Plan

Run `terraform plan` and verify that your plan succeeds and looks the same.

## Advanced areas to explore

1. Use environment variables to load your secrets.
1. Add a reference to the Public Terraform Module for [Azure Compute](https://registry.terraform.io/modules/Azure/compute/azurerm)

## Resources

- [Using Terraform Modules](https://www.terraform.io/docs/modules/usage.html)
- [Source Terraform Modiules](https://www.terraform.io/docs/modules/sources.html)
- [Public Module Registry](https://www.terraform.io/docs/registry/index.html)
