---
layout: post
title:  "Azure Blob Storage"
date:   2018-02-19 13:49:33 +0000
tags: [cloud, Azure, pentest, redteam, blueteam]
---
![](/blog/assets/win_logo.png)

This post investigates how Azure manages its blob storage, and how to create manage and list blob containers and attempt file enumeration and retrieval.

Like Google Storage, Azure Blobs are secure by default; a container and any blobs within it may only be accessed by the owner of the storage account. To give anonymous users read permissions to a container and its blobs, you can set the container permissions to allow public access. Anonymous users can read blobs within a publicly accessible container without authenticating the request.

![](/blog/assets/win_s3_perms.png)

You can configure a container with the following permissions:

* **No public read access**: The container and its blobs are private and can only be accessed by the storage account owner. This is the default for all new containers.
* **Public read access for blobs only**: Blobs within the container can be read by any anonymous Internet user. But container data is not available. Anonymous clients cannot enumerate the blobs within the container.
* **Full public read access**: All container and blob data can be read by anonymous Internet user. Clients can enumerate blobs within the container by anonymous request, but cannot enumerate containers within the storage account.

# Creating a Blob Storage Area
First create a storage account:

![](/blog/assets/win_s3_create.png)

Next Choose an appropriate Container

![](/blog/assets/win_s3_create_2.png)

# Using Azure Powershell
## List Storage Accounts
<pre>
> Get-AzureRMStorageAccount | Select StorageAccountName, Location
</pre>

![](/blog/assets/win_s3_ps_1.png)

There are additional StorageAccounts depicted, first is for a web-shell interface in Azure, the last one is to gather diagnostics from a test virtual machine.

# Using an existing Storage Account
<pre>
> $resourceGroup = “test”

> $storageAccountName = “netscyllatest”

> $storageAccount = Get-AzureRmStorageAccount -ResourceGroupName $resourceGroup -Name $storageAccountName
</pre>

![](/blog/assets/win_s3_ps_2.png)

# Delete a Storage Account
<pre>
> Remove-AzureRmStorageAccount -ResourceGroup $resourceGroup -AccountName $storageAccountName
</pre>
# List Blobs in a Container
Or list files (blobs) within a container
<pre>
> $ctx = storageAccount.Context

>Get-AzureStorageBlob -Container $ContainerName -Context $ctx | select Name
</pre>
![](/blog/assets/win_s3_ps_3.png)

Above you can see three containers:

test-blob = public blobs, private container
test-cont = public blobs, public container *
test-priv = private blobs, private container
*=container data should be available but cannot be enumerated.

# Test private blob access
To verify that you have no access to the blobs in that container, construct the URL to one of the blobs without a shared access signature and try to view the blob. Using the HTTPS protocol, the URL will be in the following format:

https://storageaccountname.blob.core.windows.net/containername/blobname

Example:

![](/blog/assets/win_s3_create_3.png)

# Conclusion
Attacking Azure blob containers is much more difficult than attacking AWS S3, and Google Storage as containers are not easily enumerated. Also to succeed in an attack; the attacker needs to know the specific name of the blob stored in the container.

# References
* https://docs.microsoft.com/en-us/azure/storage/blobs/storage-manage-access-to-resources
* https://docs.microsoft.com/en-us/azure/storage/blobs/storage-how-to-use-blobs-powershell
