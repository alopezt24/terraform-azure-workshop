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
  version             = "1.1.5"
  resource_group_name = "myapp-compute-windows"
  location            = "eastus"
  admin_password      = "ComplxP@ssw0rd!"
  vm_os_simple        = "WindowsServer"
  nb_public_ip        = 0
  vnet_subnet_id      = "${module.network.vnet_subnets[0]}"
}
```

Run a `terraform init` and `terraform plan` to verify that all the resources look correct.

- Note: when running a plan, if you see the following error, please ensure you have completed the steps in **Setup Azure Service Principal** section.
![](../../img/2018-06-07-16-23-29.png)

> Note: Take a minute to analyse why you needed to run another `terraform init` command before you could run a plan.

<details><summary>View Output</summary>
<p>

```sh
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + module.windowsservers.azurerm_availability_set.vm
      id:                                                               <computed>
      location:                                                         "eastus"
      managed:                                                          "true"
      name:                                                             "myvm-avset"
      platform_fault_domain_count:                                      "2"
      platform_update_domain_count:                                     "2"
      resource_group_name:                                              "myapp-compute"
      tags.%:                                                           <computed>

  + module.windowsservers.azurerm_network_interface.vm
      id:                                                               <computed>
      applied_dns_servers.#:                                            <computed>
      dns_servers.#:                                                    <computed>
      enable_ip_forwarding:                                             "false"
      internal_dns_name_label:                                          <computed>
      internal_fqdn:                                                    <computed>
      ip_configuration.#:                                               "1"
      ip_configuration.0.load_balancer_backend_address_pools_ids.#:     <computed>
      ip_configuration.0.load_balancer_inbound_nat_rules_ids.#:         <computed>
      ip_configuration.0.name:                                          "ipconfig0"
      ip_configuration.0.primary:                                       <computed>
      ip_configuration.0.private_ip_address:                            <computed>
      ip_configuration.0.private_ip_address_allocation:                 "dynamic"
      ip_configuration.0.public_ip_address_id:                          "${length(azurerm_public_ip.vm.*.id) > 0 ? element(concat(azurerm_public_ip.vm.*.id, list(\"\")), count.index) : \"\"}"
      ip_configuration.0.subnet_id:                                     "/subscriptions/27e9ff76-ce7b-4176-b2bb-4d3f40e1c999/resourceGroups/myapp-networking/providers/Microsoft.Network/virtualNetworks/acctvnet/subnets/subnet1"
      location:                                                         "eastus"
      mac_address:                                                      <computed>
      name:                                                             "nic-myvm-0"
      network_security_group_id:                                        "${azurerm_network_security_group.vm.id}"
      private_ip_address:                                               <computed>
      private_ip_addresses.#:                                           <computed>
      resource_group_name:                                              "myapp-compute"
      tags.%:                                                           <computed>
      virtual_machine_id:                                               <computed>

  + module.windowsservers.azurerm_network_security_group.vm
      id:                                                               <computed>
      location:                                                         "eastus"
      name:                                                             "myvm-3389-nsg"
      resource_group_name:                                              "myapp-compute"
      security_rule.#:                                                  "1"
      security_rule.0.access:                                           "Allow"
      security_rule.0.description:                                      "Allow remote protocol in from all locations"
      security_rule.0.destination_address_prefix:                       "*"
      security_rule.0.destination_port_range:                           "3389"
      security_rule.0.direction:                                        "Inbound"
      security_rule.0.name:                                             "allow_remote_3389_in_all"
      security_rule.0.priority:                                         "100"
      security_rule.0.protocol:                                         "tcp"
      security_rule.0.source_address_prefix:                            "*"
      security_rule.0.source_port_range:                                "*"
      tags.%:                                                           <computed>

  + module.windowsservers.azurerm_public_ip.vm
      id:                                                               <computed>
      domain_name_label:                                                "winsimplevmips"
      fqdn:                                                             <computed>
      ip_address:                                                       <computed>
      location:                                                         "eastus"
      name:                                                             "myvm-0-publicIP"
      public_ip_address_allocation:                                     "dynamic"
      resource_group_name:                                              "myapp-compute"
      tags.%:                                                           <computed>

  + module.windowsservers.azurerm_resource_group.vm
      id:                                                               <computed>
      location:                                                         "eastus"
      name:                                                             "myapp-compute"
      tags.%:                                                           "1"
      tags.source:                                                      "terraform"

  + module.windowsservers.azurerm_virtual_machine.vm-windows
      id:                                                               <computed>
      availability_set_id:                                              "${azurerm_availability_set.vm.id}"
      boot_diagnostics.#:                                               "1"
      boot_diagnostics.0.enabled:                                       "false"
      delete_data_disks_on_termination:                                 "false"
      delete_os_disk_on_termination:                                    "false"
      location:                                                         "eastus"
      name:                                                             "myvm0"
      network_interface_ids.#:                                          <computed>
      os_profile.#:                                                     "1"
      os_profile.249456377.admin_password:                              <sensitive>
      os_profile.249456377.admin_username:                              "azureuser"
      os_profile.249456377.computer_name:                               "myvm0"
      os_profile.249456377.custom_data:                                 <computed>
      os_profile_windows_config.#:                                      "1"
      os_profile_windows_config.429474957.additional_unattend_config.#: "0"
      os_profile_windows_config.429474957.enable_automatic_upgrades:    "false"
      os_profile_windows_config.429474957.provision_vm_agent:           "false"
      os_profile_windows_config.429474957.winrm.#:                      "0"
      resource_group_name:                                              "myapp-compute"
      storage_image_reference.#:                                        "1"
      storage_image_reference.3904372903.id:                            ""
      storage_image_reference.3904372903.offer:                         "WindowsServer"
      storage_image_reference.3904372903.publisher:                     "MicrosoftWindowsServer"
      storage_image_reference.3904372903.sku:                           "2016-Datacenter"
      storage_image_reference.3904372903.version:                       "latest"
      storage_os_disk.#:                                                "1"
      storage_os_disk.0.caching:                                        "ReadWrite"
      storage_os_disk.0.create_option:                                  "FromImage"
      storage_os_disk.0.disk_size_gb:                                   <computed>
      storage_os_disk.0.managed_disk_id:                                <computed>
      storage_os_disk.0.managed_disk_type:                              "Premium_LRS"
      storage_os_disk.0.name:                                           "osdisk-myvm-0"
      tags.%:                                                           "1"
      tags.source:                                                      "terraform"
      vm_size:                                                          "Standard_DS1_V2"

  + module.windowsservers.random_id.vm-sa
      id:                                                               <computed>
      b64:                                                              <computed>
      b64_std:                                                          <computed>
      b64_url:                                                          <computed>
      byte_length:                                                      "6"
      dec:                                                              <computed>
      hex:                                                              <computed>
      keepers.%:                                                        "1"
      keepers.vm_hostname:                                              "myvm"


Plan: 7 to add, 0 to change, 0 to destroy.
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
