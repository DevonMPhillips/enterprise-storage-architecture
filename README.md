# enterprise-storage-architecture


---

**Project Goal**
Design and implement a highly available, durable Azure Storage solution for DMP Consulting using:

Storage account redundancy (LRS / ZRS / GRS / RA-GRS)
Object replication between storage accounts
Encryption configuration
Azure Storage Explorer
AzCopy for data transfer

This project focuses on resiliency, availability, and enterprise-grade storage design.

---

**Scenario**
DMP Consulting is now supporting enterprise clients with critical data storage requirements.

The company requires:

High availability in case of region failure
Data replication between storage accounts
Secure encrypted storage by default
Efficient bulk data transfer tools
Visibility into storage structure using GUI and CLI tools

You are tasked with designing and validating this storage architecture.

---

**Architecture Overview**

<img width="1399" height="441" alt="diagram drawio" src="https://github.com/user-attachments/assets/463132a9-5cda-4ef8-b2f6-195ce7b47026" />

---

**Create Storage Accounts**

Lets go ahead and create the Primary and Secondary storage accounts

az storage account create --name dmpstorageprimary --resource-group Storage-RG --location centralus --sku Standard_LRS --kind StorageV2 --tags owner=devon department=IT && az storage account create --name dmpstoragesecondary --resource-group Storage-RG --location eastus --sku Standard_LRS --kind StorageV2 --tags owner=devon department=IT

<img width="1132" height="625" alt="image" src="https://github.com/user-attachments/assets/9c3a5fbd-ecc4-4b9e-b922-f085b8d47cce" />

Lets configure storage redundancy

LRS: 3 copies in same datacenter, cheapest, least durable
ZRS: 3 zones in same region, zone-level redundancy
GRS: Replication to secondary region, geo-redundancy
RA-GRS: Read access to secondary region, read acces in failover region

Lets change redundancy to either GRS or RA-GRS. Currently is LRS

az storage account update --name dmpstorageprimary --resource-group Storage-RG --sku "Standard_GRS"

<img width="1135" height="627" alt="image" src="https://github.com/user-attachments/assets/7143b837-8228-4086-bbd4-85cc8e6adb9a" />

Its now been updated to be GRS 

---

Object Replication

Lets first start out by creating some containers. We'll create a destination and source container.

az storage container create --name source-container --account-name dmpstorageprimary
 --auth-mode login && az storage container create --name destination-container --account-name dmpstorageprimary
 --auth-mode login

 <img width="1135" height="651" alt="image" src="https://github.com/user-attachments/assets/cacd8a8c-5ae1-4002-a93f-9dd11be624d7" />

Now that we've created the source and destination container lets configure some object replication rules.

az storage account or-policy create --resource-group storage-RG --account-name dmpstorageprimary --source-account dmpstorageprimary --destination-account dmpstoragesecondary --source-container source-container --destination-container destination-container

I got an error when i attempted to run the cmd. All i needed to do was enable the subscription feature.

1. az feature register --namespace "Microsoft.Storage" --name "EnableObjectReplicationTags"

2. az provider register --namespace "Microsoft.Storage"


<img width="1124" height="327" alt="image" src="https://github.com/user-attachments/assets/b0f6f66a-74d5-4c53-bd8f-096928d4fda4" />

Another error i ran into. Object replication requires both Change Feed and Blob Versioning to be active. Easy fix we just need to enable those settings.

<img width="1127" height="107" alt="image" src="https://github.com/user-attachments/assets/4ffc3ddc-cc6e-464c-8a4f-76ee6dcdea85" />


3. az storage account blob-service-properties update --resource-group storage-RG --account-name dmpstoragesecondary --enable-change-feed true --enable-versioning true

4. az storage account or-policy create --resource-group storage-RG --account-name dmpstoragesecondary --source-account dmpstorageprimary --destination-account dmpstoragesecondary --source-container source-container --destination-container destination-container

<img width="1130" height="710" alt="image" src="https://github.com/user-attachments/assets/9662de6e-6bf9-4039-8925-f60f526509f7" />

Now that that has been created lets test it! We'll create a test.txt file and upload it into dmpstorageprimary and its should replicate to our secondary account.

<img width="1136" height="717" alt="image" src="https://github.com/user-attachments/assets/48654e1c-6077-4077-a66a-59f5c5f1f8a8" />

<img width="1132" height="712" alt="image" src="https://github.com/user-attachments/assets/31b2e2bb-fd4e-4272-889a-a9afe7b7b76f" />

Now that the file has been uploaded to dmpstorageprimary lets check out dmpstoragesecondary.

<img width="1132" height="712" alt="image" src="https://github.com/user-attachments/assets/f3a168a5-8c23-47c0-bfd5-09a0f0cd2ad2" />

This text.txt has been successfully replicated.

---

**Configure Encryption**
Lets take a look at configuring encryption.

Azure uses Microsoft-managed keys by default. Customer Managed Keys would allow for greater control or compliance requirement scenarios. 

<img width="1132" height="712" alt="image" src="https://github.com/user-attachments/assets/36d9e566-46fb-4ea9-a4ac-29246e6528e5" />

I am going to leave the default settings.

---

Azure Storage Explorer

Azure Storage Explorer allows to you browse containers, upload files, download files, and view metadata.

---
**AzCopy**

