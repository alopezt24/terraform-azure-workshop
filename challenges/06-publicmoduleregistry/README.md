# 06 - Public Module Registry

## Expected Outcome

In this challenge, you will take a look at the public [Module Registry](https://registry.terraform.io/) and create a Virtual Machine from verified ![](../../img/2018-05-14-07-27-11.png) Public Modules.

So far we have been using Azure CLI account credentials for Terraform the Terraform Azure provider. In this challenge we will authenticate using a [Service Principal](https://www.terraform.io/docs/providers/azurerm/authenticating_via_service_principal.html) instead.

## How to

### Setup Azure Service Principal

- From the Cloud Shell, locate the correct Subscription (you may only have one) you want to create a Service Principal for:
```
az account list
```
- Note the `id` and proceed with the commands below. We will create a Service Principal with Contributor rights and save the details in a file.
```
export subscription_id=<id-from-az-account-list-command>
az account set --subscription=${subscription_id}
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/${subscription_id}" > service_principal.txt
```
- You may get the message `Retrying role assignment creation:` a few times. Once completed view the contents of `service_principal.txt` file, you should see a json file with the new credentials:
```
kawsar@Azure:~/AzureWorkChallenges/challenge06$ cat service_principal.txt
{
  "appId": "7ebbffeb-####-####-####-#############",
  "displayName": "azure-cli-2018-09-30-16-19-19",
  "name": "http://azure-cli-2018-09-30-16-19-19",
  "password": "56a6c9ca-####-####-####-#############",
  "tenant": "0e3e2e88-####-####-####-#############"
}
```

- We will set [4 environment variables](https://www.terraform.io/docs/providers/azurerm/#testing) that will allow terraform to authenticate to Azure using Service Principal credentials:
  - ARM_SUBSCRIPTION_ID - The ID of the Azure Subscription in which to run the Acceptance Tests.
  - ARM_CLIENT_ID - The Client ID of the Service Principal.
  - ARM_CLIENT_SECRET - The Client Secret associated with the Service Principal.
  - ARM_TENANT_ID - The Tenant ID to use.

- Issue the commands below to export these variables in your Azure Cloud shell:
```
export ARM_SUBSCRIPTION_ID=${subscription_id}
export ARM_CLIENT_ID=$(cat service_principal.txt | jq -r .appId)
export ARM_CLIENT_SECRET=$(cat service_principal.txt | jq -r .password)
export ARM_TENANT_ID=$(cat service_principal.txt | jq -r .tenant)
```
- Verify the above variables are set correctly: `env | grep ARM_`. You should see the above 4 variables set with Service Principal attributes. If so, we are all set to proceed with the remaining steps in this challenge!
  - If you see an empty output, please rerun the export commands above.
  - If any of the values are empty, please check the contents of `service_principal.txt` file.

### Navigate the Public Module Registry

Open your browser and navigate to the [Module Registry](https://registry.terraform.io/).

Search for "Compute" which will yield all compute resources in the registry.

Now Filter By 'azurerm' which should give you (among others) the Microsoft Azure Compute Module.

If you are having issues locating the module, you can find it directly at [https://registry.terraform.io/modules/Azure/compute/azurerm/latest](https://registry.terraform.io/modules/Azure/compute/azurerm/latest).

Search again for "Networking" and apply the same Filter By, which should give you the Microsoft Azure Networking Module.

If you are having issues locating the module, you can find it directly at [https://registry.terraform.io/modules/Azure/network/azurerm/latest](https://registry.terraform.io/modules/Azure/network/azurerm/latest).

### Create Terraform Configuration

From the Cloud Shell, change directory into a folder specific to this challenge. If you created the scaffolding in Challenge 00, then then you can use the command `cd ~/AzureWorkChallenges/challenge06/`.

To create an Azure Virtual Machine we need the networking in place, to do so we will be using both the modules above.
The Networking module will create the Virtual Network and Subnet, then the Compute module will use that subnet as an input to create its Virtual Machine.

Create a `main.tf` file in this directory and add the networking module.

The following will configure the module to create a single Virtual Network and a Subnet.

```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "example" {
  name     = "myapp-networking"
  location = "eastus"
}

module "network" {
  source              = "Azure/network/azurerm"
  version             = "3.5.0"
  resource_group_name = azurerm_resource_group.example.name

tags = {
    environment = "dev"
  }

depends_on = [azurerm_resource_group.example]
}
```

Run a `terraform init` and `terraform plan` to verify that all the resources look correct.

<details><summary>View Output</summary>
<p>

```sh
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
    + create
 <= read (data resources)

Terraform will perform the following actions:

  # azurerm_resource_group.example will be created
  + resource "azurerm_resource_group" "example" {
      + id       = (known after apply)
      + location = "eastus"
      + name     = "myapp-networking"
    }

  # module.network.data.azurerm_resource_group.network will be read during apply
  # (config refers to values not yet known)
 <= data "azurerm_resource_group" "network"  {
      + id       = (known after apply)
      + location = (known after apply)
      + name     = "myapp-networking"
      + tags     = (known after apply)

      + timeouts {
          + read = (known after apply)
        }
    }

  # module.network.azurerm_subnet.subnet[0] will be created
  + resource "azurerm_subnet" "subnet" {
      + address_prefix                                 = (known after apply)
      + address_prefixes                               = [
          + "10.0.1.0/24",
        ]
      + enforce_private_link_endpoint_network_policies = false
      + enforce_private_link_service_network_policies  = false
      + id                                             = (known after apply)
      + name                                           = "subnet1"
      + resource_group_name                            = "myapp-networking"
      + service_endpoints                              = []
      + virtual_network_name                           = "acctvnet"
    }

  # module.network.azurerm_virtual_network.vnet will be created
  + resource "azurerm_virtual_network" "vnet" {
      + address_space         = [
          + "10.0.0.0/16",
        ]
      + dns_servers           = []
      + guid                  = (known after apply)
      + id                    = (known after apply)
      + location              = (known after apply)
      + name                  = "acctvnet"
      + resource_group_name   = "myapp-networking"
      + subnet                = (known after apply)
      + tags                  = {
          + "environment" = "dev"
        }
      + vm_protection_enabled = false
    }

Plan: 3 to add, 0 to change, 0 to destroy.
```

</p>
</details>

Run `terraform apply` to create the infrastructure.

### View Outputs

The Public Registry contains a lot of information about the module. Navigate to the outputs tab for the [Networking Module](https://registry.terraform.io/modules/Azure/network/azurerm/latest).

We can see the outputs we should expect and a short description of each of them.

### Add Compute Module - Windows

With Networking in place you can now add the Compute module to create a Windows Virtual Machine.

```hcl
module "windowsservers" {
  source              = "Azure/compute/azurerm"
  version             = "3.14.0"
  resource_group_name = azurerm_resource_group.example.name
  admin_password      = "ComplxP@ssw0rd!"
  vm_os_simple        = "WindowsServer"
  nb_public_ip        = 0
  vnet_subnet_id      = module.network.vnet_subnets[0]

depends_on = [azurerm_resource_group.example]
}
```

Run a `terraform init` and `terraform plan` to verify that all the resources look correct.

- Note: when running a plan, if you see the following error, please ensure you have completed the steps in **Setup Azure Service Principal** section.

> Note: Take a minute to analyse why you needed to run another `terraform init` command before you could run a plan.

<details><summary>View Output</summary>
<p>

```sh
Terraform detected the following changes made outside of Terraform since the last "terraform apply":

  # module.network.azurerm_subnet.subnet[0] has been changed
  ~ resource "azurerm_subnet" "subnet" {
        id                     = "/subscriptions/.../resourceGroups/myapp-net working/providers/Microsoft.Network/virtualNetworks/acctvnet/subnets/subnet1"
        name                   = "subnet1"
      + service_endpoint_policy_ids = []
        # (7 unchanged attributes hidden)
    }
  # module.network.azurerm_virtual_network.vnet has been changed
  ~ resource "azurerm_virtual_network" "vnet" {
        id                    = "/subscriptions/.../resourceGroups/myapp-networking/providers/Microsoft.Network/virtualNetworks/acctvnet"
        name                  = "acctvnet"
      ~ subnet                = [
          + {
              + address_prefix = "10.0.1.0/24"
              + id             = "/subscriptions/.../resourceGroups/myapp-networking/providers/Microsoft.Network/virtualNetworks/acctvnet/subnets/subnet1"
              + name           = "subnet1"
              + security_group = ""
            },
        ]
        tags                  = {
            "environment" = "dev"
        }
        # (6 unchanged attributes hidden)
    }
  # azurerm_resource_group.example has been changed
  ~ resource "azurerm_resource_group" "example" {
        id       = "/subscriptions/.../resourceGroups/myapp-networking"
        name     = "myapp-networking"
      + tags     = {}
        # (1 unchanged attribute hidden)
    }

Unless you have made equivalent changes to your configuration, or ignored the relevant attributes using ignore_changes, the following
plan may include actions to undo or respond to these changes.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
symbols:
  + create

Terraform will perform the following actions:

  # module.windowsservers.azurerm_availability_set.vm will be created
  + resource "azurerm_availability_set" "vm" {
      + id                           = (known after apply)
      + location                     = "eastus"
      + managed                      = true
      + name                         = "myvm-avset"
      + platform_fault_domain_count  = 2
      + platform_update_domain_count = 2
      + resource_group_name          = "myapp-networking"
      + tags                         = {
          + "source" = "terraform"
        }
    }

  # module.windowsservers.azurerm_network_interface.vm[0] will be created
  + resource "azurerm_network_interface" "vm" {
      + applied_dns_servers           = (known after apply)
      + dns_servers                   = (known after apply)
      + enable_accelerated_networking = false
      + enable_ip_forwarding          = false
      + id                            = (known after apply)
      + internal_dns_name_label       = (known after apply)
      + internal_domain_name_suffix   = (known after apply)
      + location                      = "eastus"
      + mac_address                   = (known after apply)
      + name                          = "myvm-nic-0"
      + private_ip_address            = (known after apply)
      + private_ip_addresses          = (known after apply)
      + resource_group_name           = "myapp-networking"
      + tags                          = {
          + "source" = "terraform"
        }
      + virtual_machine_id            = (known after apply)

      + ip_configuration {
          + name                          = "myvm-ip-0"
          + primary                       = (known after apply)
          + private_ip_address            = (known after apply)
          + private_ip_address_allocation = "dynamic"
          + private_ip_address_version    = "IPv4"
          + subnet_id                     = "/subscriptions/.../resourceGroups/myapp-networking/providers/Microsoft.Network/virtualNetworks/acctvnet/subnets/subnet1"
        }
    }

  # module.windowsservers.azurerm_network_interface_security_group_association.test[0] will be created
  + resource "azurerm_network_interface_security_group_association" "test" {
      + id                        = (known after apply)
      + network_interface_id      = (known after apply)
      + network_security_group_id = (known after apply)
    }

  # module.windowsservers.azurerm_network_security_group.vm will be created
  + resource "azurerm_network_security_group" "vm" {
      + id                  = (known after apply)
      + location            = "eastus"
      + name                = "myvm-nsg"
      + resource_group_name = "myapp-networking"
      + security_rule       = (known after apply)
      + tags                = {
          + "source" = "terraform"
        }
    }

  # module.windowsservers.azurerm_virtual_machine.vm-windows[0] will be created
  + resource "azurerm_virtual_machine" "vm-windows" {
      + availability_set_id              = (known after apply)
      + delete_data_disks_on_termination = false
      + delete_os_disk_on_termination    = false
      + id                               = (known after apply)
      + license_type                     = (known after apply)
      + location                         = "eastus"
      + name                             = "myvm-vmWindows-0"
      + network_interface_ids            = (known after apply)
      + resource_group_name              = "myapp-networking"
      + tags                             = {
          + "source" = "terraform"
        }
      + vm_size                          = "Standard_D2s_v3"

      + boot_diagnostics {
          + enabled = false
        }

      + identity {
          + identity_ids = (known after apply)
          + principal_id = (known after apply)
          + type         = (known after apply)
        }

      + os_profile {
          + admin_password = (sensitive value)
          + admin_username = "azureuser"
          + computer_name  = "myvm-0"
          + custom_data    = (known after apply)
        }

      + os_profile_windows_config {
          + enable_automatic_upgrades = false
          + provision_vm_agent        = true
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
          + sku       = "2019-Datacenter"
          + version   = "latest"
        }

      + storage_os_disk {
          + caching                   = "ReadWrite"
          + create_option             = "FromImage"
          + disk_size_gb              = (known after apply)
          + managed_disk_id           = (known after apply)
          + managed_disk_type         = "Premium_LRS"
          + name                      = "myvm-osdisk-0"
          + os_type                   = (known after apply)
          + write_accelerator_enabled = false
        }
    }

  # module.windowsservers.random_id.vm-sa will be created
  + resource "random_id" "vm-sa" {
      + b64_std     = (known after apply)
      + b64_url     = (known after apply)
      + byte_length = 6
      + dec         = (known after apply)
      + hex         = (known after apply)
      + id          = (known after apply)
      + keepers     = {
          + "vm_hostname" = "myvm"
        }
    }

Plan: 6 to add, 0 to change, 0 to destroy.
```

</p>
</details>

Before applying, take a look at all the resources that are going to be created from our simple `module` block.

Run `terraform apply` to create the infrastructure.

### Clean up

Run `terraform destroy` to remove everything we created.

## Advanced areas to explore

1. Add a public ip to the Windows Compute instance using additional parameters built into the Compute module.
1. Create a second module instance of Compute to stand up a Linux Virtual Machine.

## Resources

- [Azurerm Networking Moduel Source](https://github.com/Azure/terraform-azurerm-network)
- [Azurerm Compute Module Source](https://github.com/Azure/terraform-azurerm-compute)
- [Network Security Groups](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-nsg)
