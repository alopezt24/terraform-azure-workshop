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

    sas_token = "UQzuvHgRYdPOjsKYl8Fj4MzdQuNYHsvIZqFJqD9Ij/rfUX+x/7GhfLwkZhJJiztkNGG6e1MNb1IpC0AuBD7SCA=="
  }
}
```

Run `terraform init`.

### View Lock State

Run a plan and view the file in the portal, notice how a lease is put on it.
