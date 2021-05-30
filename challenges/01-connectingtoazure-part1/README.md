# 01 - Connection To Azure

## Expected Outcome

In this challenge, you will use Terraform from the Azure Cloud Shell to create simple infrastructure in your Azure Subscription.

In this challenge, you will:

- Initialize Terraform
- Run a `plan` on simple a simple resource
- Run an `apply` to create Azure infrastructure
- Run a `destroy` to remove Azure infrastructure

## How To

### Create Terraform Configuration

From the Cloud Shell, change directory into a folder specific to this challenge. If you created the scaffolding in Challenge 00, then then you can use the command `cd ~/AzureWorkChallenges/challenge01/`.

Create a file named `main.tf` and add a single Resource Group resource.

```hcl
# We strongly recommend using the required_providers block to set the
# Azure Provider source and version being used
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=2.61.0"
    }
  }
}

# Configure the Microsoft Azure Provider
provider "azurerm" {
  features {}
}

# Create a resource group
resource "azurerm_resource_group" "test" {
  name     = "challenge01-rg"
  location = "eastus"
}
```

This will create a simple Resource Group and allow you to walk through the Terraform Workflow.

### Run the Terraform Workflow

`terraform init`
<details><summary>View Output</summary>
<p>

```sh
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/azurerm...
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

</p>
</details>

---
`terraform plan`

<details><summary>View Output</summary>
<p>

```sh
$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # azurerm_resource_group.test will be created
  + resource "azurerm_resource_group" "test" {
      + id       = (known after apply)
      + location = "eastus"
      + name     = "challenge01-rg"
    }


Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```

</p>
</details>

---
`terraform apply`
<details><summary>View Output</summary>
<p>

```sh
$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + azurerm_resource_group.main
      id:       <computed>
      location: "eastus"
      name:     "challenge01-rg"
      tags.%:   <computed>


Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

azurerm_resource_group.main: Creating...
  location: "" => "eastus"
  name:     "" => "challenge01-rg"
  tags.%:   "" => "<computed>"
azurerm_resource_group.main: Creation complete after 1s (ID: /subscriptions/.../resourceGroups/challenge01-rg)

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```
</p>
</details>

---

Congrats, you just created your first Azure resource using Terraform!

### Verify in the Azure Portal

Head over to the [Azure Portal](https://portal.azure.com/)

View all Resource Groups and you should see the recently created Resource Group.
![](../../img/2018-05-09-10-20-28.png)

### Scale Resources

Now add a new Resource Group resource that scales with a `count` parameter.

```hcl
resource "azurerm_resource_group" "count" {
  name     = "challenge01-rg-${count.index}"
  location = "eastus"
  count    = 2
}
```

Run another `terraform plan` and `terraform apply`.

### Cleanup Resources

When you are done, destroy the infrastructure, we no longer need it.

```sh
$ terraform destroy
azurerm_resource_group.main: Refreshing state... (ID: /subscriptions/.../resourceGroups/challenge01-rg)
azurerm_resource_group.count[0]: Refreshing state... (ID: /subscriptions/.../resourceGroups/challenge01-rg-0)
azurerm_resource_group.count[1]: Refreshing state... (ID: /subscriptions/.../resourceGroups/challenge01-rg-1)

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  - azurerm_resource_group.count[0]

  - azurerm_resource_group.count[1]

  - azurerm_resource_group.main


Plan: 0 to add, 0 to change, 3 to destroy.

Do you really want to destroy?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

azurerm_resource_group.main: Destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg)
azurerm_resource_group.count[1]: Destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg-1)
azurerm_resource_group.count[0]: Destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg-0)
azurerm_resource_group.count.0: Still destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg-0, 10s elapsed)
azurerm_resource_group.main: Still destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg, 10s elapsed)
azurerm_resource_group.count.1: Still destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg-1, 10s elapsed)
azurerm_resource_group.count.1: Still destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg-1, 20s elapsed)
azurerm_resource_group.count.0: Still destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg-0, 20s elapsed)
azurerm_resource_group.main: Still destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg, 20s elapsed)
azurerm_resource_group.count.1: Still destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg-1, 30s elapsed)
azurerm_resource_group.count.0: Still destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg-0, 30s elapsed)
azurerm_resource_group.main: Still destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg, 30s elapsed)
azurerm_resource_group.count.0: Still destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg-0, 40s elapsed)
azurerm_resource_group.count.1: Still destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg-1, 40s elapsed)
azurerm_resource_group.main: Still destroying... (ID: /subscriptions/.../resourceGroups/challenge01-rg, 40s elapsed)
azurerm_resource_group.main: Destruction complete after 47s
azurerm_resource_group.count[0]: Destruction complete after 47s
azurerm_resource_group.count[1]: Destruction complete after 47s

Destroy complete! Resources: 3 destroyed.
```
---

### Cleanup

Because the infrastructure is now managed by Terraform, we can destroy just like before.

Run a `terraform destroy` and follow the prompts to remove the infrastructure.

## Advanced areas to explore

1. Play around with adjusting the `count` and `name` parameters, then running `plan` and `apply`.
1. Run the `plan` command with the `-out` option and apply that output.
1. Add tags to each resource.

## Resources

- [Terraform Count](https://www.terraform.io/docs/configuration/interpolation.html#count-information)
