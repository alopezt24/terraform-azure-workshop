# 01 - Connection To Azure - Part 2

This is a continuation of Challenge 01 where we will learn how to use Terraform imports. For part 1 please visit the following link:

[01 - Connection To Azure](01 - Connection To Azure). 

We will stay in the same working directory for this part of the challenge: 

```bash
~/AzureWorkChallenges/challenge01/
```

## Expected Outcome

In this challenge, you will use the `terraform import` command to import existing Azure Resources into the Terraform state file.

In this challenge, you will:
- Create 2 Resources in Azure Portal
- Create corresponding Terraform configuration
- Run `terraform import` to update State file and verify using `terraform plan`
- Make a change in Terraform configuration
- Run a `plan` and `apply` to update Azure infrastructure
- Run a `destroy` to remove Azure infrastructure

## How To - Part 2 - Import Resources

### Create Infrastructure in the Portal

Navigate to the Azure Portal and click on the "Resource groups" item on the left side and then click  "+ Add":

![](../../img/2018-05-28-13-58-49.png)

In the Resource Group create blade give the resource group the name "myportal-rg" and click "Create":

![](../../img/2018-05-28-14-01-30.png)

Once the Resource Group is created, navigate to it.

Find the "+ Add" button and click it:

![](../../img/2018-05-28-14-03-05.png)

Search for "Storage Account" and click the first item and then click "Create" :

![](../../img/2018-05-28-14-04-39.png)


In the Storage Account create blade, fill out the following:

- Name = Must be a unique name, there will be a green checkmark that shows up in the text box if your name is available. Example "<YOURUSERNAME>storageaccount"
- Storage Account Kind: **Storage (General Purpose V1)**
- Replication = LRS
- **Resource Group = Use Existing and select "myportal-rg"**
- Require Secure transfer should be set to: “disabled”

![](../../img/2018-05-28-14-05-39.png)

Click "Create"

At this point we have a Resource Group and a Storage Account and are ready to import this into Terraform.

![](../../img/2018-05-28-14-09-39.png)

### Create Terraform Configuration

Your Azure Cloud Shell should still be in the folder for Challenge 01 with a single `main.tf` file.
- First delete this file so we can start from scratch.
```
rm main.tf
```
- Now create a new `main.tf` file to proceed with the subsequent steps.

We have two resources we need to import into our Terraform Configuration, to do this we need to do two things:

1. Create the base Terraform configuration for both resources.
2. Run `terraform import` to bring the infrastructure into our state file.

To create the base configuration place the following code into the `main.tf` file.

```hcl
resource "azurerm_resource_group" "main" {
  name     = "myportal-rg"
  location = "centralus"
}

resource "azurerm_storage_account" "main" {
  name                     = "myusernamestorageaccount"
  resource_group_name      = "${azurerm_resource_group.main.name}"
  location                 = "centralus"
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

```

`terraform plan`

Shows 2 to add

```sh
Terraform will perform the following actions:

  + azurerm_resource_group.main
      id:                               <computed>
      location:                         "centralus"
      name:                             "myportal-rg"
      tags.%:                           <computed>

  + azurerm_storage_account.main
      id:                               <computed>
      access_tier:                      <computed>
      account_encryption_source:        "Microsoft.Storage"
      account_kind:                     "Storage"
      account_replication_type:         "LRS"
      account_tier:                     "Standard"
      enable_blob_encryption:           <computed>
      enable_file_encryption:           <computed>
      location:                         "centralus"
      name:                             "myusernamestorageaccount"
      primary_access_key:               <computed>
      primary_blob_connection_string:   <computed>
      primary_blob_endpoint:            <computed>
      primary_connection_string:        <computed>
      primary_file_endpoint:            <computed>
      primary_location:                 <computed>
      primary_queue_endpoint:           <computed>
      primary_table_endpoint:           <computed>
      resource_group_name:              "myportal-rg"
      secondary_access_key:             <computed>
      secondary_blob_connection_string: <computed>
      secondary_blob_endpoint:          <computed>
      secondary_connection_string:      <computed>
      secondary_location:               <computed>
      secondary_queue_endpoint:         <computed>
      secondary_table_endpoint:         <computed>
      tags.%:                           <computed>


Plan: 2 to add, 0 to change, 0 to destroy.
```

> CAUTION: This is not what we want!

### Import the Resource Group

We need two values to run the `terraform import` command:

1. Resource Address from our configuration
1. Azure Resource ID

The Resource Address is simple enough, based on the configuration above it is simply "azurerm_resource_group.main".

The Azure Resource ID can be retrieved using the Azure CLI by running `az group show -g myportal-rg --query id`. The value should look something like "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myportal-rg".

Now run the import command:
- Note: exclude the quotes ("") when running `terraform import`.

```sh
$ terraform import azurerm_resource_group.main /subscriptionsxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myportal-rg

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```

### Import the Storage Account

The process here is the same.

The Resource Address is simple enough, based on the configuration above it is simply "azurerm_storage_account.main".

The Azure Resource ID can be retrieved using the Azure CLI by running `az storage account show -g myportal-rg -n myusernamestorageaccount --query id`. The value should look something like "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myportal-rg/providers/Microsoft.Storage/storageAccounts/myusernamestorageaccount".

```sh
$ terraform import azurerm_storage_account.main /subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myportal-rg/providers/Microsoft.Storage/storageAccounts/myusernamestorageaccount

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```

### Verify Plan

Run a `terraform plan`, you should see "No changes. Infrastructure is up-to-date.".

### Make a Change

Add the following tag configuration to both the Resource Group and the Storage Account:

```hcl
resource "azurerm_resource_group" "main" {
  ...
  tags {
    terraform = "true"
  }
}

resource "azurerm_storage_account" "main" {
  ...
  tags {
    terraform = "true"
  }
}
```

Run a plan, we should see two changes.

```sh
  ~ azurerm_resource_group.main
      tags.%:         "0" => "1"
      tags.terraform: "" => "true"

  ~ azurerm_storage_account.main
      tags.%:         "0" => "1"
      tags.terraform: "" => "true"


Plan: 0 to add, 2 to change, 0 to destroy.
```

Apply those changes.

SUCCESS! You have now brought existing infrastructure into Terraform.

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
