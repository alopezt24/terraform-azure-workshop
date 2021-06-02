# 07 - Remote Backend

## Expected Outcome

In this challenge, you will move your state file to a remote backend.

## How to

### Create Azure Storage Account

In the Portal, create a SA.

Get the account information, including SAS token.

Create a blob container

### Config Backend

Update your configuration with the info:

```hcl
terraform {
  backend "azurerm" {
    storage_account_name = "remotebackend"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
    sas_token            = "?sv=2020-02-10&ss=bfqt&srt=co&sp=rwdlacuptfx&se=2021-06-02T11:07:35Z&st=2021-06-02T03:07:35Z&spr=https&si$
}
}
```

Run `terraform init -reconfigure`.

### View Lock State

Run a plan and view the file in the portal, notice how a lease is put on it.
