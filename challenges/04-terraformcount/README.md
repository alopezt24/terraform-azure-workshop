# 04 - Terraform Count

## Expected Outcome

In this challenge, you will take what you did in Challenge 03 and expand to take a count variable.

Be aware that if you did not destroy the infrastructure from Challenge 03 you may run into resource naming conflicts (namely "Resource already exists").

## How to

### Copy Terraform Configuration

From the Cloud Shell, change directory into a folder specific to this challenge. If you created the scaffolding in Challenge 00, then then you can use the command `cd ~/AzureWorkChallenges/challenge04/`.

Copy the `main.tf` file from challenge 03 into the current directory.

Be sure to update the value of the `name` variable:

```hcl
variable "name" {
  default = "challenge04"
}
```

### Add a count variable

Create a new variable called `vmcount` and default it to `1`.

```hcl
variable "vmcount" {
  default = 1
}
```

### Update Existing Resources

Not every resource needs to scale as the number of VM's increase.

The Resource Group, Virtual Network and Subnet will not change.

We will have to scale the Network Interface, Public IP, and VM resources.

Each of these resources you will need make these changes:

- Add the count variable

```hcl
    count = "${var.vmcount}"
```

- Update the resource name to include the count index, for example the VM resource:

```hcl
    name = "${var.name}-vm${count.index}"
```

- Update the OS Disk Name on the VM resource:

```hcl
    storage_os_disk {
        name = "${var.name}vm${count.index}-osdisk"
        ...
    }
```

- Update the computer name on the VM resource.

```hcl
    os_profile {
        computer_name = "${var.name}vm${count.index}"
        ...
    }
```

- Update the ID reference for the Virtual Machine:

```hcl
    network_interface_ids = ["${element(azurerm_network_interface.main.*.id, count.index)}"]
```

- Update the Public IP ID reference for the Network Interface:

```hcl
    public_ip_address_id = "${element(azurerm_public_ip.main.*.id, count.index)}"
```

- Update the Private IP outputs to display an array of IPs:

```hcl
    value = "${azurerm_network_interface.main.*.private_ip_address}"
```

- Update the Public IP outputs to display an array of IPs:

```hcl
    value = "${azurerm_public_ip.main.*.ip_address}"
```

### Run a plan

If all the changes above have been made without error, runnning `terraform init` and `terraform plan` should execute without any errors.

The plan should end with:

```sh
Plan: 6 to add, 0 to change, 0 to destroy.
```

Now investigate this plan in more detail and you will notice that names have been set using the count index:

```sh
...
  + azurerm_virtual_machine.module[0]
      ...
      name:                                 "challenge04-vm0"
      ...
      os_profile.3613624746.computer_name:  "challenge04vm0"
      ...

```

### Update Count

Set the default value of the count to `2`.
Before running a plan consider the following questions:

- How many resources do expect the plan to show?
- What will the outputs look like?

Your plan should look similar to the following
<details><summary>View Output</summary>
<p>

```sh
$ terraform plan
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # azurerm_network_interface.main[0] will be created
  + resource "azurerm_network_interface" "main" {
      + applied_dns_servers           = (known after apply)
      + dns_servers                   = (known after apply)
      + enable_accelerated_networking = false
      + enable_ip_forwarding          = false
      + id                            = (known after apply)
      + internal_dns_name_label       = (known after apply)
      + internal_domain_name_suffix   = (known after apply)
      + location                      = "eastus"
      + mac_address                   = (known after apply)
      + name                          = "challenge04-nic"
      + private_ip_address            = (known after apply)
      + private_ip_addresses          = (known after apply)
      + resource_group_name           = "challenge04-rg"
      + virtual_machine_id            = (known after apply)

      + ip_configuration {
          + name                          = "config1"
          + primary                       = (known after apply)
          + private_ip_address            = (known after apply)
          + private_ip_address_allocation = "dynamic"
          + private_ip_address_version    = "IPv4"
          + public_ip_address_id          = (known after apply)
          + subnet_id                     = (known after apply)
        }
    }

  # azurerm_network_interface.main[1] will be created
  + resource "azurerm_network_interface" "main" {
      + applied_dns_servers           = (known after apply)
      + dns_servers                   = (known after apply)
      + enable_accelerated_networking = false
      + enable_ip_forwarding          = false
      + id                            = (known after apply)
      + internal_dns_name_label       = (known after apply)
      + internal_domain_name_suffix   = (known after apply)
      + location                      = "eastus"
      + mac_address                   = (known after apply)
      + name                          = "challenge04-nic"
      + private_ip_address            = (known after apply)
      + private_ip_addresses          = (known after apply)
      + resource_group_name           = "challenge04-rg"
      + virtual_machine_id            = (known after apply)

      + ip_configuration {
          + name                          = "config1"
          + primary                       = (known after apply)
          + private_ip_address            = (known after apply)
          + private_ip_address_allocation = "dynamic"
          + private_ip_address_version    = "IPv4"
          + public_ip_address_id          = (known after apply)
          + subnet_id                     = (known after apply)
        }
    }

  # azurerm_public_ip.main[0] will be created
  + resource "azurerm_public_ip" "main" {
      + allocation_method       = "Static"
      + fqdn                    = (known after apply)
      + id                      = (known after apply)
      + idle_timeout_in_minutes = 4
      + ip_address              = (known after apply)
      + ip_version              = "IPv4"
      + location                = "eastus"
      + name                    = "challenge04-pubip"
      + resource_group_name     = "challenge04-rg"
      + sku                     = "Basic"
    }

  # azurerm_public_ip.main[1] will be created
  + resource "azurerm_public_ip" "main" {
      + allocation_method       = "Static"
      + fqdn                    = (known after apply)
      + id                      = (known after apply)
      + idle_timeout_in_minutes = 4
      + ip_address              = (known after apply)
      + ip_version              = "IPv4"
      + location                = "eastus"
      + name                    = "challenge04-pubip"
      + resource_group_name     = "challenge04-rg"
      + sku                     = "Basic"
    }

  # azurerm_resource_group.main will be created
  + resource "azurerm_resource_group" "main" {
      + id       = (known after apply)
      + location = "eastus"
      + name     = "challenge04-rg"
    }

  # azurerm_subnet.main will be created
  + resource "azurerm_subnet" "main" {
      + address_prefix                                 = (known after apply)
      + address_prefixes                               = [
          + "10.0.1.0/24",
        ]
      + enforce_private_link_endpoint_network_policies = false
      + enforce_private_link_service_network_policies  = false
      + id                                             = (known after apply)
      + name                                           = "challenge04-subnet"
      + resource_group_name                            = "challenge04-rg"
      + virtual_network_name                           = "challenge04-vnet"
    }

  # azurerm_virtual_machine.main[0] will be created
  + resource "azurerm_virtual_machine" "main" {
      + availability_set_id              = (known after apply)
      + delete_data_disks_on_termination = false
      + delete_os_disk_on_termination    = false
      + id                               = (known after apply)
      + license_type                     = (known after apply)
      + location                         = "eastus"
      + name                             = "challenge04-vm0"
      + network_interface_ids            = (known after apply)
      + resource_group_name              = "challenge04-rg"
      + vm_size                          = "Standard_A2_v2"

      + identity {
          + identity_ids = (known after apply)
          + principal_id = (known after apply)
          + type         = (known after apply)
        }

      + os_profile {
          + admin_password = (sensitive value)
          + admin_username = "testadmin"
          + computer_name  = "challenge04vm0"
          + custom_data    = (known after apply)
        }

      + os_profile_windows_config {
          + enable_automatic_upgrades = false
          + provision_vm_agent        = false
        }

      + storage_data_disk {
          + caching                   = (known after apply)
          + create_option             = (known after apply)
          + disk_size_gb              = (known after apply)
          + lun                       = (known after apply)
          + managed_disk_id           = (known after apply)
          + managed_disk_type         = (known after apply)
          + name                      = (known after apply)
          + vhd_uri                   = (known after apply)
          + write_accelerator_enabled = (known after apply)
        }

      + storage_image_reference {
          + offer     = "WindowsServer"
          + publisher = "MicrosoftWindowsServer"
          + sku       = "2016-Datacenter"
          + version   = "latest"
        }

      + storage_os_disk {
          + caching                   = "ReadWrite"
          + create_option             = "FromImage"
          + disk_size_gb              = (known after apply)
          + managed_disk_id           = (known after apply)
          + managed_disk_type         = "Standard_LRS"
          + name                      = "challenge04vm0-osdisk"
          + os_type                   = (known after apply)
          + write_accelerator_enabled = false
        }
    }

  # azurerm_virtual_machine.main[1] will be created
  + resource "azurerm_virtual_machine" "main" {
      + availability_set_id              = (known after apply)
      + delete_data_disks_on_termination = false
      + delete_os_disk_on_termination    = false
      + id                               = (known after apply)
      + license_type                     = (known after apply)
      + location                         = "eastus"
      + name                             = "challenge04-vm1"
      + network_interface_ids            = (known after apply)
      + resource_group_name              = "challenge04-rg"
      + vm_size                          = "Standard_A2_v2"

      + identity {
          + identity_ids = (known after apply)
          + principal_id = (known after apply)
          + type         = (known after apply)
        }

      + os_profile {
          + admin_password = (sensitive value)
          + admin_username = "testadmin"
          + computer_name  = "challenge04vm1"
          + custom_data    = (known after apply)
        }

      + os_profile_windows_config {
          + enable_automatic_upgrades = false
          + provision_vm_agent        = false
        }

      + storage_data_disk {
          + caching                   = (known after apply)
          + create_option             = (known after apply)
          + disk_size_gb              = (known after apply)
          + lun                       = (known after apply)
          + managed_disk_id           = (known after apply)
          + managed_disk_type         = (known after apply)
          + name                      = (known after apply)
          + vhd_uri                   = (known after apply)
          + write_accelerator_enabled = (known after apply)
        }

      + storage_image_reference {
          + offer     = "WindowsServer"
          + publisher = "MicrosoftWindowsServer"
          + sku       = "2016-Datacenter"
          + version   = "latest"
        }

      + storage_os_disk {
          + caching                   = "ReadWrite"
          + create_option             = "FromImage"
          + disk_size_gb              = (known after apply)
          + managed_disk_id           = (known after apply)
          + managed_disk_type         = "Standard_LRS"
          + name                      = "challenge04vm1-osdisk"
          + os_type                   = (known after apply)
          + write_accelerator_enabled = false
        }
    }

  # azurerm_virtual_network.main will be created
  + resource "azurerm_virtual_network" "main" {
      + address_space         = [
          + "10.0.0.0/16",
        ]
      + guid                  = (known after apply)
      + id                    = (known after apply)
      + location              = "eastus"
      + name                  = "challenge04-vnet"
      + resource_group_name   = "challenge04-rg"
      + subnet                = (known after apply)
      + vm_protection_enabled = false
    }

Plan: 9 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + private-ip = [
      + (known after apply),
      + (known after apply),
    ]
  + public-ip  = [
      + (known after apply),
      + (known after apply),
    ]

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

</p>
</details>

Run `terraform apply` to create all the infrastructure.

### Azure Portal

In the Azure Portal, view all the resources in the `challenge04-rg` Resource Group.

How important do think naming is?

How can Terraform help with keeping things consistent?

### Clean up

Run `terraform destroy` to remove everything we created.

## Advanced areas to explore

1. Add an Azure Load Balancer.
1. Add tags to the Virtual Machine and then use the `-target` option to target only a single resource.

## Resources

- [Azure Load Balancer](https://www.terraform.io/docs/providers/azurerm/r/loadbalancer.html)
- [Resource Targeting](https://www.terraform.io/docs/commands/plan.html#resource-targeting)
