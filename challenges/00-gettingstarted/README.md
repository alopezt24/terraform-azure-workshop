# 00 - Getting Started

## Expected Outcome

In this challenge, you will connect to the [Azure Cloud Shell](https://azure.microsoft.com/en-us/features/cloud-shell/) that will be needed for future challenges.

In this challenge, you will:

- Login to the Azure Portal
- Verify `az` installation
- Verify `terraform` installation
- Create a folder structure to complete challenges

> Note: If you would rather complete the challenges from you local worskstation, detailed instructions can be found [here](local.md).
## How to

### Login to the Azure Portal

Navigate to [https://portal.azure.com](https://portal.azure.com) and login with your Azure Credentials.

This workshop will require that you have access to an Azure Subscription with at least Contributor rights to create resources. If you do not currently have access you can create a trial account by going to [https://azure.microsoft.com/en-us/free](https://azure.microsoft.com/en-us/free) and registering for a 3-month trail.

Signing up for a trial requires:

- A unique Microsoft Live Account that has not registered for a trial for in the past
- A Credit Card, used to verify identity and will not be charged unless you opt-in after the trial is over

> If you are having issues with this access, please alert the instructor ASAP as this will prevent you from completing the challenges.

### Open the Cloud Shell

Located at the top of the page is the button open the Azure Cloud Shell inside the Azure Portal.

> Note: Another option is to use the full screen Azure Cloud Shell at [https://shell.azure.com/](https://shell.azure.com/).

The first time you connect to the Azure Cloud Shell you will be prompted to setup an Azure File Share that you will persist the environment.

Click the "Bash (Linux)" option.

Select the Azure Subscription and click "Create storage".

After a few seconds you should see that your storage account has been created.

> Note: Behind the scenes this is creating a new Resource Group with the name `cloud-shell-storage-eastus` (or which ever region you defaulted to). If you need more information, it can be found [here](https://docs.microsoft.com/en-us/azure/cloud-shell/persisting-shell-storage).

SUCCESS!
You are now logged into the Azure Cloud Shell which uses your portal session to automatically authenticate you with the Azure CLI and Terraform.

### Verify Utilities

In the Cloud Shell type the following commands and verify that the utilities are installed:

`az -v`

<details><summary>View Output</summary>
<p>

```sh
$ az -v
azure-cli (2.24.0)

.
.
.

Python location '/opt/az/bin/python3'
Extensions directory '/home/tstraub/.azure/cliextensions'

Python (Linux) 3.6.1 (default, May 18 2018, 04:21:17)
[GCC 5.4.0 20160609]

Legal docs and information: aka.ms/AzureCliLegal
```
</p>
</details>

`terraform -v`

<details><summary>View Output</summary>
<p>

```sh
$ terraform -v
Terraform v0.15.4
```

</p>
</details>

### Verify Subscription

Run the command `az account list -o table`.

```sh
az account list -o table
Name                             CloudName    SubscriptionId                        State    IsDefault
-------------------------------  -----------  ------------------------------------  -------  -----------
Visual Studio Premium with MSDN  AzureCloud   xxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx  Enabled  True
Another sub1                     AzureCloud   xxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx  Enabled  False
Another sub2                     AzureCloud   xxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx  Enabled  False
Another sub3                     AzureCloud   xxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx  Enabled  False
```

If you have more than subscription, make sure that subscription is set as default using the subscription name:

```sh
az account set -s 'Visual Studio Premium with MSDN'
```

### Create Challenge Folder Structure

To make things easy for the challenges, let's create a folder structure to hold the terraform configuration we will create.

Make sure you are in the home directory:

```sh
cd ~/
```

Run the following in the azure cloud shell, this will simply create a folder structure for you to place your Terraform configuration:

```sh
mkdir AzureWorkChallenges && cd AzureWorkChallenges && mkdir challenge01 && mkdir challenge02 && mkdir challenge03 && mkdir challenge04 && mkdir challenge05 && mkdir challenge06 && mkdir challenge07
```

And if you happen to be doing this in Powershell, here's the Powershell version of that command line:
```powershell
mkdir AzureWorkChallenges; cd AzureWorkChallenges; mkdir challenge01; mkdir challenge02; mkdir challenge03; mkdir challenge04; mkdir challenge05; mkdir challenge06; mkdir challenge07
```

What you should end up with is a structure like this:

```sh
AzureWorkChallenges
|- challenge01
|- challenge02
|- challenge03
|- challenge04
|- challenge05
|- challenge06
|- challenge07
```
