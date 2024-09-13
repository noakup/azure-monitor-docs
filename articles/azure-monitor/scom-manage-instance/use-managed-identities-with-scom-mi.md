---
ms.assetid: 
title: Use Managed identities for Azure with Azure Monitor SCOM Managed Instance
description: This article describes how to use Managed identities for Azure with Azure Monitor SCOM Managed Instance.
author: PriskeyJeronika-MS
ms.author: v-gjeronika
manager: jsuri
ms.date: 05/24/2024
ms.custom: UpdateFrequency.5
ms.service: system-center
ms.subservice: operations-manager-managed-instance
ms.topic: article
---

# Use Managed identities for Azure with Azure Monitor SCOM Managed Instance

A common challenge when building cloud applications is how to securely manage the credentials in your code for authenticating various services without saving them locally on a developer workstation or in source control. 

*Managed identities for Azure* solve this problem for all your resources in Azure Active Directory by providing them with automatically managed identities. You can use a service's identity to authenticate any service that supports Azure Active Directory authentication, including Key vault, without sorting any credentials in your code. 

>[!Note]
>- *Managed identities for Azure* are the new name for the service formerly known as Managed Service Identity (MSI).
>- *Managed identities for Azure resources* are free with Azure Active Directory for Azure subscriptions. There's no extra cost.  

## Concepts 

Managed identities for Azure are based on several key concepts: 

- **Client ID** - A unique identifier generated by Azure Active Directory that is tied to an application and service principal during its initial provisioning. For more information, see   [Application (client) ID](/azure/active-directory/develop/developer-glossary#application-client-id).

- **Principal ID** - The object ID of the service principal object for your managed identity that is used to grant role-based access to an Azure resource. 

- **Service Principal** - An Azure Active Directory object, which represents the projection of an Azure Active Directory application on a given tenant. For more information, see  [Service principal](/azure/active-directory/develop/developer-glossary#service-principal-object).

## Managed identity types

There are two types of managed identities: 

- **System-assigned managed identity**: Enabled directly on an Azure service instance. The lifecycle of a system-assigned identity is unique to the Azure service instance that it's enabled on. 

- **User-assigned managed identity**: Created as a standalone Azure resource. The identity can be assigned to one or more Azure service instances and is managed separately from the lifecycles of those instances. 

For more information on Managed identity types, see  [How do managed identities for Azure resources work?](/azure/active-directory/managed-identities-azure-resources/overview#managed-identity-types). 

## Supported scenarios for SCOM Managed Instance

SCOM Managed Instance supports both System Assigned Managed Identity and User Assigned Managed Identity for the SCOM Managed Instances deployed in Azure. SCOM Managed Instance creates other dependency resources like Virtual Machine Scale Sets (VMSS) cluster to host management servers. SCOM Managed Instance onboarded the Managed identities with HOBO v2 so that the assigned Identity will be delegated to the underlying infrastructure for authenticating with sink resources. These Identities are used to authenticate other Azure services in different scenarios. 

- System Assigned Managed Identity

     - SCOM Managed Instance will send different health or performance metrics to Geneva Cluster services, monitoring the instance behavior at run time. The system Assigned Identity, which is delegated to SCOM Managed Instance resource, will be used to authenticate with Azure Geneva Cluster services. 

- User Assigned Managed Identity 

     - For SCOM Managed Instance, a Managed Identity will replace the traditional four System Center Operations Manager service accounts and will be used to access the SQL Managed Instance database. SCOM Managed Instance reads/writes customer workload monitoring data to SQL managed instance databases. The User Assigned Identity assigned to SCOM Managed Instance resource will be used for authenticating from System Center Operations Manager servers to SQL Managed instance. 

     - SCOM Managed Instance onboarding process takes the domain user credentials stored in Customer Key vault. The secrets in the Customer Key vault are accessed using the managed identity assigned to SCOM Managed Instance. 

     During SCOM Managed Instance onboarding, you must provide the User Managed Identity, which has access to Customer Key vault and SQL Managed Instance.

## Create a Managed Service Identity (MSI)

Create a [Managed Service identity](/system-center/scom/create-operations-manager-managed-instance?view=sc-om-2022&tabs=prereqs-portal#create-a-managed-service-identity-msi&preserve-view=true) and provide it with the right access level on Azure resource.

## Create a Key vault and add Credentials as a secret in the Key vault  

Store the domain account you create in the Active Directory in a Key vault account for security. Azure Key Vault is a cloud service that provides a secure store for keys, secrets, and certificates. For more information, see [Azure Key vault](/azure/key-vault/general/overview).

- Create a [Key vault](/azure/key-vault/general/quick-create-portal).
- Create a [Secret for the domain account](/azure/key-vault/secrets/quick-create-portal).
- Assign a Reader role on this key vault to the Managed Service Identity (MSI). For more information, see  [Assign a Key vault access policy](/azure/key-vault/general/assign-access-policy?tabs=azure-portal).

## Set the Active Directory Admin value in the SQL Managed Instance

[Set the Active Directory Admin value](/system-center/scom/create-operations-manager-managed-instance?view=sc-om-2022&tabs=prereqs-portal#set-the-active-directory-admin-value-in-the-sql-mi&preserve-view=true) in the SQL Managed Instance.