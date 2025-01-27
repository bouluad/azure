# Azure Services for Building and Managing OS Images with On-Premise GitHub Runners

## Overview

This guide describes the Azure services required and the steps to enable developer teams to create their own GitHub workflows to build OS images. The OS image creation happens in a centralized CI subscription managed by your team and is pushed to the respective developer team's Azure subscription. The GitHub runners used for this process are hosted on-premise.

## Azure Services Involved

1. Azure Image Builder

  Purpose: Simplifies the creation of custom images.
  
  Usage: Utilized for building OS images with custom configurations, such as adding a private CA certificate bundle and a predefined user.

2. Azure Shared Image Gallery

  Purpose: Distributes and shares OS images across subscriptions and regions.
  
  Usage: Stores the built images and makes them accessible to developer team subscriptions.

3. Azure Managed Images

  Purpose: Stores a single VM image in a subscription.
  
  Usage: Temporary storage of images before pushing to the Shared Image Gallery.

4. Azure Service Principal

  Purpose: Provides secure access to Azure resources from the on-premise GitHub runners.
  
  Usage: Each team uses a dedicated service principal to authenticate and interact with Azure resources in their subscription.

5. Azure Role-Based Access Control (RBAC)

  Purpose: Manages permissions for service principals and other Azure resources.
  
  Usage: Assigns appropriate roles (e.g., Contributor or Image Builder Owner) to service principals.

6. Azure Virtual Network (VNet)

  Purpose: Ensures secure communication between on-premise runners and Azure resources.
  
  Usage: The CI subscription has a dedicated VNet with subnets configured for Azure Image Builder and other necessary resources.

7. Azure Storage Account

  Purpose: Stores logs, scripts, and artifacts during the image-building process.
  
  Usage: Used by Azure Image Builder to access configuration scripts and store build logs.

8. Azure Key Vault

  Purpose: Secures sensitive information such as service principal credentials.
  
  Usage: Stores and manages secrets required for authentication and automation.

## Workflow Steps

### Step 1: Prepare the CI Subscription
Create a dedicated subscription (CI subscription) for building OS images.

Set up a Shared Image Gallery in the CI subscription.

Configure a VNet with a dedicated subnet for Azure Image Builder.

Create a storage account to hold scripts and logs.

Set up an Azure Key Vault to store service principal credentials.

### Step 2: Configure Service Principals

Create a service principal for each developer team.

Assign the Contributor role to the service principal for their respective subscriptions.

Grant the service principal access to the Shared Image Gallery in the CI subscription.

Store service principal credentials in the Azure Key Vault.

### Step 3: Set Up On-Premise GitHub Runners

Configure the on-premise runners to securely connect to Azure resources via the CI subscription VNet.

Install necessary tools on the runners (e.g., Packer, Azure CLI).

### Step 4: Developer Workflow with GitHub Actions

Developers create their own GitHub workflows to build OS images:

The workflow uses the service principal credentials to authenticate with Azure.

Packer runs on the on-premise GitHub runner to create the OS image:

Copies the private CA bundle into the image.

Adds a predefined user (e.g., afyaut) for automation purposes.

The image is stored temporarily as a Managed Image in the CI subscription.

The image is copied to the Shared Image Gallery in the CI subscription.

The image is replicated to the developer team's subscription using az image copy.

### Step 5: Assigning Permissions and Access

Ensure the service principal has appropriate RBAC permissions to:

Access the Shared Image Gallery.

Push images to their respective subscriptions.

Use Azure Policies to enforce compliance (e.g., images must be built via the CI subscription).
