---
lab:
  title: "Implement Microsoft Dev Box"
  module: "Implement Developer Self-Service"
---

# Lab 01: Implement Microsoft Dev Box

## Estimated timing: 60 minutes

## Prerequisites

- A Microsoft Entra tenant with 3 pre-created user accounts (and, optionally 3 pre-created Microsoft Entra groups) representing 3 different roles involved in Microsoft Dev Box deployments. For the sake of clarity, the user and group names in the lab instructions will be matching the information in the following table:

| User              | Group                        | Role                  |
| ----------------- | ---------------------------- | --------------------- |
| platformegineer01 | DevCenter_Platform_Engineers | Platform engineer     |
| devlead01         | DevCenter_Dev_Leads          | Development team lead |
| devuser01         | DevCenter_Dev_Users          | Developer             |

- An Azure subscription to host Microsoft Dev Box resources associated with the Microsoft Entra tenant hosting the user and group accounts
- A Microsoft Intune subscription associated with the same Microsoft Entra tenant as the Azure subscription
- A Microsoft Intune license assigned to the three pre-created Microsoft Entra user accounts
- A GitHub repo created as a fork of https://github.com/microsoft/devcenter-catalog
- A GitHub repo created as a fork of https://github.com/dhruvchand/contoso-co-eShop

## Objectives

After completing this workshop, you will be able to:

- Implement a Microsoft Dev Box environment
- Customize a Microsoft Dev Box environment

## Exercise 1: Implement a Microsoft Dev Box environment

In this exercise, you will leverage a set of features provided by Microsoft to implement a Microsoft Dev Box environment. This approach focuses on minimizing the effort involved in building a functional developer self-service solution.

The exercise consists of the following tasks:

- Task 1: Create a dev center
- Task 2: Review the dev center settings
- Task 3: Create a dev box definition
- Task 4: Create a project
- Task 5: Create a dev box pool
- Task 6: Configure permissions
- Task 7: Evaluate a dev box

### Task 1: Create a dev center

In this task, you will create an Azure dev center that will be used throughout the first exercise of this lab. A dev center is a platform engineering service that centralizes the creation and management of scalable, pre-configured development and deployment environments, optimizing collaboration and resource utilization for software development teams.

1. Connect to the console session of the lab computer and, if needed, sign in by using the local Administrator account.
1. Within the console session to the lab computer, start a web browser and navigate to the Azure portal at `https:\\portal.azure.com`.
1. When prompted to authenticate, sign in by using the Microsoft Entra user account provided in the lab environment.
1. In the Azure portal, in the **Search** text box, search for and select **Dev centers**.
1. On the **Dev centers** page, select **+ Create**.
1. On the **Basics** tab of the **Create a dev center** page, specify the following settings and then select **Next: Settings**:

   | Setting                                                                 | Value                                                        |
   | ----------------------------------------------------------------------- | ------------------------------------------------------------ |
   | Subscription                                                            | The name of the Azure subscription you are using in this lab |
   | Resource group                                                          | The name of a **new** resource group **rg-devcenter-01**     |
   | Name                                                                    | **devcenter-01**                                             |
   | Location                                                                | **(US) East US**                                             |
   | Attach a quick start catalog - Azure deployment environment definitions | Enabled                                                      |
   | Attach a quick start catalog - Dev box customization tasks              | Disabled                                                     |

   > **Note:** You will configure a catalog with customization tasks enabled in the second exercise of this lab.

1. On the **Settings** tab of the **Create a dev center** page, specify the following settings and then select **Review + Create**:

   | Setting                                    | Value   |
   | ------------------------------------------ | ------- |
   | Enable catalogs per project                | Enabled |
   | Allow Microsoft hosted network in projects | Enabled |
   | Enable Azure Monitor agent installation    | Enabled |

   > **Note:** By design, resources from catalogs attached to the dev center are available to all projects within it. The setting **Enable catalogs per project** makes possible to attach additional catalogs to arbitrarily selected projects as well.

   > **Note:** Dev boxes can be connected either to a virtual network in your own Azure subscription or a Microsoft hosted one, depending on whether they need to communicate with resources in your environment. If there is no such need, by enabling the setting **Allow Microsoft hosted network in projects**, you introduce the option to connect dev boxes to a Microsoft hosted network, effectively minimizing the management and configuration overhead.

   > **Note:** The setting **Enable Azure Monitor agent installation** automatically triggers installation of the Azure Monitor agent on all dev boxes in the dev center.

1. On the **Review + Create** tab, wait for the validation to complete and then select **Create**.

   > **Note:** Wait for the project to be provisioned. The project creation might take about 1 minute.

1. On the **Deployment is completed** page, select **Go to resource**.

### Task 2: Review the dev center settings

In this task, you will review the basic configuration setting of the dev center you created in the previous task.

1. Within the console session to the lab computer, in the web browser displaying the Azure portal, on the **devcenter-01** page, in the vertical navigation menu on the left side, expand the **Environment configuration** section and select **Catalogs**.
1. On the **devcenter-01 \| Catalogs** page, notice that the dev center is configured with the **quickstart-environment-definitions** catalog, which points to the GitHub repository `https://github.com/microsoft/devcenter-catalog.git`.
1. Verify that the **Status** column contains the **Sync successful** entry. If that is not the case, use the following sequence of steps to re-create the catalog:

   1. Select the checkbox next to the autogenerated catalog entry **quickstart-environment-definitions** and then, in the toolbar, select **Delete**.
   1. On the **devcenter-01 \| Catalogs** page, select **+ Add**.
   1. In the **Add catalog** pane, in the **Name** text box, enter **quickstart-environment-definitions-fix**, in the **Catalog source** section, select **GitHub**, in the **Authentication type**, select **GitHub app**, leave the checkbox **Automatically sync this catalog** checkbox enabled, and then select **Sign in with GitHub**.
   1. In the **Sign in with GitHub** window, enter the GitHub credentials provided in the lab environment and select **Sign in**.

      > **Note:** These GitHub credentials provide you with access to a GitHub repo created as a fork of https://github.com/microsoft/devcenter-catalog

   1. When prompted, in the **Authorize Microsoft DevCenter** window, select **Authorize Microsoft DevCenter**.
   1. Back in the **Add catalog** pane, in the **Repo** drop-down list, select **devcenter-catalog**, in the **Branch** drop-down list, accept the **Default branch** entry, in the **Folder path**, enter **Environment-Definitions** and then select **Add**.
   1. Back on the **devcenter-01 \| Catalogs** page, verify that the sync completes successfully by monitoring the entry in the **Status** column.

1. On the **devcenter-01 \| Catalogs** page, select the **quickstart-environment-definitions-fix** entry.
1. On the **quickstart-environment-definitions-fix** page, review the list of predefined environment definitions.

   > **Note:** Each entry represents a definition of an Azure deployment environment defined in a respective subfolder of the **Environment-Definitions** folder of the GitHub repository `https://github.com/microsoft/devcenter-catalog.git`.

   > **Note:** A deployment environment is a collection of Azure resources defined in a template referred to as an environment definition. Developers can use these definitions to deploy infrastructure that will serve to host their solutions. For more information regarding Azure deployment environments, refer to the Microsoft Learn article [What is Azure Deployment Environments?](https://learn.microsoft.com/en-us/azure/deployment-environments/overview-what-is-azure-deployment-environments)

### Task 3: Create a dev box definition

In this task, you will create a dev box definition. Its purpose is to define the operating system, tools, settings, and resources that serve as a blueprint for creating consistent and tailored development environments (referred to as dev boxes).

1. Within the console session to the lab computer, in the web browser displaying the Azure portal, on the **devcenter-01** page, in the vertical navigation menu on the left side, expand the **Dev box configuration** section and select **Dev box definitions**.
1. On the **devcenter-01 \| Dev box definitions** page, select **+ Create**.
1. On the **Create dev box definition** page, specify the following settings and then select **Create**:

   | Setting            | Value                                                                                |
   | ------------------ | ------------------------------------------------------------------------------------ | ----------------------- |
   | Name               | **devbox-definition-01**                                                             |
   | Image              | \*\*Visual Studio 2022 Enterprise on Windows 11 Enterprise + Microsoft 365 Apps 24H2 | Hibernate supported\*\* |
   | Image version      | **Latest**                                                                           |
   | Compute            | **8 vCPU, 32 GB RAM**                                                                |
   | Storage            | **256 GB SSD**                                                                       |
   | Enable hibernation | Enabled                                                                              |

   > **Note:** Wait for the dev box definition to be created. This should take less than 1 minute.

### Task 4: Create a dev center project

In this task, you will create a dev center project. A dev center project typically corresponds with a development project within your organization. For example, you might create a project for the development of a line of business application, and another project for the development of the company website. All projects in a dev center share the same dev box definitions, network connection, catalogs, and compute galleries. You might consider creating multiple dev center projects if you have multiple development projects that have separate project administrators and access permissions requirements.

1. Within the console session to the lab computer, in the web browser displaying the Azure portal, on the **devcenter-01** page, in the vertical navigation menu on the left side, expand the **Manage** section and select **Projects**.
1. On the **devcenter-01 \| Projects** page, select **+ Create**.
1. On the **Basics** tab of the **Create a project** page, specify the following settings and then select **Next: Dev box management**:

   | Setting        | Value                                                        |
   | -------------- | ------------------------------------------------------------ |
   | Subscription   | The name of the Azure subscription you are using in this lab |
   | Resource group | **rg-devcenter-01**                                          |
   | Dev center     | **devcenter-01**                                             |
   | Name           | **devcenter-project-01**                                     |
   | Description    | **devcenter-project-01**                                     |

1. On the **Dev box management** tab of the **Create a project** page, specify the following settings and then select **Next: Catalogs**:

   | Setting                 | Value |
   | ----------------------- | ----- |
   | Enable dev box limits   | Yes   |
   | Dev boxes per developer | **2** |

1. On the **Catalogs** tab of the **Create a project** page, specify the following settings and then select **Review + Create**:

   | Setting                            | Value   |
   | ---------------------------------- | ------- |
   | Deployment environment definitions | Enabled |
   | Image definitions                  | Enabled |

   > **Note:** Since you enabled project level catalogs when creating the dev center, for catalogs attached to a dev project, these settings determine which catalog items are imported to the project upon sync.

1. On the **Review + Create** tab of the **Create a dev center** page, select **Create**:

   > **Note:** Wait for the project to be created. This should take less than 1 minute.

1. On the **Deployment is completed** page, select **Go to resource**.

### Task 5: Create a dev box pool

In this task, you will create a dev box pool in the dev center project you created in the previous task. Dev box pools are used by dev box users to create dev boxes. A dev box pool links a dev box definition with a network connection. In general, you can choose from Microsoft-hosted connections or your own Azure network connections. The network connection determines the location of a dev box is hosted and its access to other cloud and on-premises resources. Consider creating a dev box pool with a network connection nearest the dev box users. In addition, to reduce the cost of running dev boxes, you can configure a dev box pool to shut them down daily at a predefined time.

1. Within the console session to the lab computer, in the web browser displaying the Azure portal, on the **devcenter-project-01** page, in the vertical navigation menu on the left side, expand the **Manage** section and select **Dev box pools**.
1. On the **devcenter-project-01 \| Dev box pools** page, select **+ Create**.
1. On the **Basics** tab of the **Create a dev box pool** page, specify the following settings and then select **Create**:

   | Setting                                                                                                           | Value                                    |
   | ----------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
   | Name                                                                                                              | **devcenter-project-01-devbox-pool-01**  |
   | Definition                                                                                                        | **devbox-definition-01**                 |
   | Network connection                                                                                                | **Deploy to a Microsoft hosted network** |
   | Region                                                                                                            | **(US) East US**                         |
   | Enable single sign-on                                                                                             | Enabled                                  |
   | Dev box Creator Privileges                                                                                        | **Local Administrator**                  |
   | Enable auto-stop on schedule                                                                                      | Enabled                                  |
   | Stop time                                                                                                         | **07:00 PM**                             |
   | Time zone                                                                                                         | Your current time zone                   |
   | Enable hibernate on disconnect                                                                                    | Enabled                                  |
   | Grace period in minutes                                                                                           | **60**                                   |
   | I confirm that my organization has Azure Hybrid Benefits licenses, which will apply to all dev boxes in this pool | Enabled                                  |

   > **Note:** Wait for the dev box pool to be created. This might take about 2 minutes.

### Task 6: Configure permissions

In this task, you will assign suitable Microsoft dev box-related permissions to the three Microsoft Entra security principals which have been provisioned in your lab environment. These security principals correspond to typical roles in platform engineering scenarios:

| User              | Group                        | Role                  |
| ----------------- | ---------------------------- | --------------------- |
| platformegineer01 | DevCenter_Platform_Engineers | Platform engineer     |
| devlead01         | DevCenter_Dev_Leads          | Development team lead |
| devuser01         | DevCenter_Dev_Users          | Developer             |

Microsoft dev box relies on Azure role-based access control (Azure RBAC) to control access to project-level functionality. Platform engineers should have full control to create and manage dev centers, their catalogs, and projects. This effectively requires the owner or contributor role, depending on whether they also need the ability to delegate permissions to others. Development team leads should be assigned the dev center Project Admin role, which grants the ability to perform administrative tasks on Microsoft Dev Box projects. Dev box users need to ability to create and manage their own dev boxes, which are associated with the Dev Box User role.

> **Note:** You will start by assigning permissions to the Microsoft Entra group intended to contain platform engineer user accounts.

1. Within the console session to the lab computer, in the web browser displaying the Azure portal, navigate to the **devcenter-01** page and, in the vertical navigation menu on the left side, select **Access control (IAM)**.
1. On the **devcenter-01 \| Access control (IAM)** page, select **+ Add** and, in the drop-down list, select **Add role assignment**.
1. On the **Role** tab of the **Add role assignment** page, select the **Privileged administrator role** tab, in the list of roles, select **Owner** and finally select **Next**.
1. On the **Members** tab of the **Add role assignment** page, ensure that the **User, group, or service principal** option is selected and click **+ Select members**.
1. In the **Select members** pane, search for and select **DevCenter_Platform_Engineers** and then click **Select**.
1. Back on the **Members** tab of the **Add role assignment** page, select **Next**.
1. On the **Conditions** tab of the **Add role assignment** page, in the **What user can do** section, select the option **Allow user to assign all roles (highly privileged)** and then select **Next**.
1. On the **Review + assign** tab of the **Add role assignment** page, select **Review + assign**.

   > **Note:** Next you will assign permissions to the Microsoft Entra group intended to contain development team lead user accounts.

1. Back on the **devcenter-01 \| Access control (IAM)** page, in the vertical navigation menu on the left side, expand the **Manage** section, select **Projects**, and, in the list of projects, select **devcenter-project-01**.
1. On the **devcenter-project-01** page, in the vertical navigation menu on the left side, select **Access control (IAM)**.
1. On the **devcenter-project-01 \| Access control (IAM)** page, select **+ Add** and, in the drop-down list, select **Add role assignment**.
1. On the **Role** tab of the **Add role assignment** page, ensure that the **Job function roles** tab is selected, in the list of roles, select **DevCenter Project Admin** and select **Next**.
1. On the **Members** tab of the **Add role assignment** page, ensure that the **User, group, or service principal** option is selected and click **+ Select members**.
1. In the **Select members** pane, search for and select **DevCenter_Dev_Leads** and then click **Select**.
1. Back on the **Members** tab of the **Add role assignment** page, select **Next**.
1. On the **Review + assign** tab of the **Add role assignment** page, select **Review + assign**.

   > **Note:** Finally, you will assign permissions to the Microsoft Entra group intended to contain developer user accounts.

1. Back on the **devcenter-project-01 \| Access control (IAM)** page, select **+ Add** and, in the drop-down list, select **Add role assignment**.
1. On the **Role** tab of the **Add role assignment** page, ensure that the **Job function roles** tab is selected, in the list of roles, select **DevCenter Dev Box Users** and select **Next**.
1. On the **Members** tab of the **Add role assignment** page, ensure that the **User, group, or service principal** option is selected and click **+ Select members**.
1. In the **Select members** pane, search for and select **DevCenter_Dev_Users** and then click **Select**.
1. Back on the **Members** tab of the **Add role assignment** page, select **Next**.
1. On the **Review + assign** tab of the **Add role assignment** page, select **Review + assign**.

### Task 7: Evaluate a dev box

In this task, you will evaluate a dev box functionality by using a Microsoft Entra developer user account.

1. Within the console session to the lab computer, start a web browser incognito/in-private and navigate to the Microsoft Dev Box developer portal at `https://aka.ms/devbox-portal`.
1. When prompted to sign in, provide the credentials of the **devuser01** user account.
1. On the **Welcome, devuser01** page of the Microsoft Dev Box developer portal, select **+ New dev box**.
1. In the **Add a dev box** pane, in the **Name** text box, enter **devuser01box01**
1. Review other information presented in the **Add a dev box** pane, including the project name, dev box pool specifications, hibernation support status, and the scheduled shutdown timing. In addition, note the option to apply customizations and the notification that dev box creation might take up to 65 minutes.

   > **Note:** Dev box names must be unique within a project.

1. In the **Add a dev box** pane, select **Create**.

   > **Note:** Do not wait for the dev box to be created. Instead, proceed directly to the next exercise of this lab. However, consider returning to this exercise and stepping through the rest of this task once you complete the next exercise.

1. Once the dev box is fully provisioned and running, connect to it by selecting the option to **Connect via app**.

   > **Note:** Connectivity to a dev box can be established by using a Remote Desktop Windows app, a Remote Desktop client (mstsc.exe), or directly within a web browser window.

1. In the pop-up window titled **This site is trying to open Microsoft Remote Connection Center**, select **Open**. This will automatically initiate a Remote Desktop session to the dev box.
1. When prompted for credentials, authenticate by providing the user name and the password of the **devuser01** account.
1. Within the Remote Desktop session to the dev box, verify that its configuration includes an installation of Visual Studio 2022 and Microsoft 365 apps.

   > **Note:** You can shut down the dev box directly from the Microsoft Dev Box developer portal as a dev user by first selecting the ellipsis symbol in the **Your dev box** interface and then selecting **Shut down** from the cascading menu. Alternatively, as a platform engineer or development team leads, you can control dev box lifecycle from the **Dev box pools** section of the corresponding dev center project.

## Exercise 2: Customize a Microsoft Dev Box environment

In this exercise, you will customize the functionality of the Microsoft Dev Box environment. This approach focuses on the extent of changes you can apply when implementing a custom developer self-service solution.

The exercise consists of the following tasks:

- Task 1: Create an Azure compute gallery and attach it to the dev center
- Task 2: Configure authentication and authorization for Azure Image Builder
- Task 3: Create a custom image by using Azure Image Builder
- Task 4: Create an Azure dev center network connection
- Task 5: Adding image definitions to an Azure dev center project
- Task 6: Create a customized dev box pool
- Task 7: Evaluate a customized dev box

### Task 1: Create an Azure compute gallery and attach it to the dev center

In this task, you will create an Azure compute gallery and attach it to the dev center you created in the previous exercise of this lab. A gallery is a repository residing in an Azure subscription, which helps you build structure and organization around custom images. After you attach a compute gallery to a dev center and populate it with images, you will be able to create dev box definitions based on images stored in the compute gallery.

1. If needed, connect to the console session of the lab computer and sign in by using the local Administrator account.
1. Within the console session to the lab computer, start a web browser and navigate to the Azure portal at `https:\\portal.azure.com`.
1. When prompted to authenticate, sign in by using the Microsoft Entra user account provided in the lab environment.
1. In the Azure portal, in the **Search** text box, search for and select **Azure compute galleries**.
1. On the **Azure compute galleries** page, select **+ Create**.
1. On the **Basics** tab of the **Create Azure compute gallery** page, specify the following settings and then select **Next: Sharing method**:

   | Setting        | Value                                                        |
   | -------------- | ------------------------------------------------------------ |
   | Subscription   | The name of the Azure subscription you are using in this lab |
   | Resource group | **rg-devcenter-01**                                          |
   | Name           | **compute_gallery_01**                                       |
   | Region         | **(US) East US**                                             |

1. On the **Sharing method** tab of the **Create Azure compute gallery** page, ensure that the **Role based access control (RBAC)** option is selected and then select **Review + Create**:
1. On the **Review + Create** tab, wait for the validation to complete and then select **Create**.

   > **Note:** Wait for the project to be provisioned. The Azure compute gallery creation should take less than 1 minute.

1. In the Azure portal, search for and select **Dev centers** and, on the **Dev centers** page, select **devcenter-01**.
1. On the **devcenter-01** page, in the vertical navigation menu on the left side, expand the **Dev box configuration** section and select **Azure compute galleries**.
1. On the **devcenter-01 \| Azure compute galleries** page, select **+ Add**.
1. In the **Add Azure compute gallery** pane, in the **Gallery** drop-down list, select **compute_gallery_01** and then select **Add**.

### Task 2: Configure authentication and authorization

In this task, you will create a user-assigned managed identity that will be used by Azure Image Builder to add images to the Azure compute gallery you created in the previous task. You will also configure the required permissions by creating a custom role based access control (RBAC) role and assigning it to the managed identity. This will allow you to use Azure Image Builder in the next task to build a custom image.

1. In the Azure portal, select the **Cloud Shell** toolbar icon to open the Cloud Shell pane and, if needed, select **Switch to PowerShell** to start a PowerShell session and, in the **Switch to PowerShell in Cloud Shell** dialog box, select **Confirm**.

   > **Note:** If this is the first time you are opening Cloud Shell, in the **Welcome to Azure Cloud Shell** dialog box, select **PowerShell**, in the **Getting Started** pane, select the option **No Storage Account required** and, in the **Subscription** drop-down list, select the name of the Azure subscription you are using in this lab.

1. In the PowerShell session of the Cloud Shell pane, run the following commands to ensure that all required resource providers are registered:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
   Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
   Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
   Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
   Register-AzResourceProvider -ProviderNamespace Microsoft.Network
   ```

1. Run the following command to install the required PowerShell modules (when prompted, type **A** and press the **Enter** key):

   ```powershell
   'Az.ImageBuilder', 'Az.ManagedServiceIdentity' | ForEach-Object {Install-Module -Name $_ -AllowPrerelease}
   ```

1. Run the following commands to set up variables that will be referenced throughout the image build process:

   ```powershell
   $currentAzContext = Get-AzContext
   # the target Azure subscription ID
   $subscriptionID=$currentAzContext.Subscription.Id
   # the target Azure resource group name
   $imageResourceGroup='rg-devcenter-01'
   # the target Azure region
   $location='eastus'
   # the reference name assigned to the image created by using the Azure Image Builder service
   $runOutputName="aibWinImg01"
   # image template name
   $imageTemplateName="templateWinVSCode01"
   # the Azure compute gallery name
   $computeGallery = 'compute_gallery_01'
   ```

1. Run the following commands to create a user-assigned managed identity (VM Image Builder uses the user identity you provide to store images in the target Azure Compute Gallery):

   ```powershell
   # Install the Azure PowerShell module to support AzUserAssignedIdentity
   Install-Module -Name Az.ManagedServiceIdentity
   # Generate a pseudo-random integer to be used for resource names
   $timeInt=$(get-date -UFormat "%s")

   # Create an identity
   $identityName='identityAIB' + $timeInt
   New-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName -Location $location
   $identityNameResourceId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id
   $identityNamePrincipalId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId
   ```

1. Run the following commands to grant the newly created user-assigned managed identity the permissions required to store images in the **rg-devcenter-01** resource group:

   ```powershell
   # Set variables
   $imageRoleDefName = 'Custom Azure Image Builder Image Def' + $timeInt
   $aibRoleImageCreationUrl = 'https://raw.githubusercontent.com/azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json'
   $aibRoleImageCreationPath = 'aibRoleImageCreation.json'

   # Customize the role definition file
   Invoke-WebRequest -Uri $aibRoleImageCreationUrl -OutFile $aibRoleImageCreationPath -UseBasicParsing
   ((Get-Content -path $aibRoleImageCreationPath -Raw) -Replace '<subscriptionID>', $subscriptionID) | Set-Content -Path $aibRoleImageCreationPath
   ((Get-Content -path $aibRoleImageCreationPath -Raw) -Replace '<rgName>', $imageResourceGroup) | Set-Content -Path $aibRoleImageCreationPath
   ((Get-Content -path $aibRoleImageCreationPath -Raw) -Replace 'Azure Image Builder Service Image Creation Role', $imageRoleDefName) | Set-Content -Path $aibRoleImageCreationPath

   # Create a role definition
   New-AzRoleDefinition -InputFile  ./aibRoleImageCreation.json

   # Assign the role to the VM Image Builder user-assigned managed identity within the scope of the **rg-devcenter-01** resource group
   New-AzRoleAssignment -ObjectId $identityNamePrincipalId -RoleDefinitionName $imageRoleDefName -Scope "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup"
   ```

### Task 3: Create a custom image by using Azure Image Builder

In this task, you will use Azure Image Builder to create a custom image based on an existing Azure Resource Manager (ARM) template that defines a Windows 11 Enterprise image with automatically installed Choco and Visual Studio Code. Azure VM Image Builder considerably simplifies the process of defining and provisioning VM images. It relies on an image configuration that you specify to configure an automated imaging pipeline. Subsequently, developers will be able to use such images to provision their dev boxes.

1. In the PowerShell session of the Cloud Shell pane, run the following commands to create an image definition to be added to the Azure compute gallery you created in the first task of this exercise:

   ```powershell
   # ensure that the image definition security type property is set to 'TrustedLaunch'
   $securityType = @{Name='SecurityType';Value='TrustedLaunch'}
   $features = @($securityType)
   # Image definition name
   $imageDefName = 'imageDefDevBoxVSCode01'

   # Create the image definition
   New-AzGalleryImageDefinition -GalleryName $computeGallery -ResourceGroupName $imageResourceGroup -Location $location -Name $imageDefName -OsState generalized -OsType Windows -Publisher 'Contoso' -Offer 'vscodedevbox' -Sku '1-0-0' -Feature $features -HyperVGeneration 'V2'
   ```

   > **Note:** A dev box image must satisfy a number of requirements including the use of Generation 2, Hyper-V v2, and Windows 10 or 11 Enterprise version 20H2 or later. For their full list, refer to the Microsoft Learn article [Configure Azure Compute Gallery for Microsoft Dev Box](https://learn.microsoft.com/en-us/azure/dev-box/how-to-configure-azure-compute-gallery).

1. Run the following commands to create an empty file named template.json that will contain an ARM template defining a Windows 11 Enterprise image with automatically installed Choco and Visual Studio Code:

   ```powershell
   Set-Location -Path ~
   $templateFile = 'template.json'
   Set-Content -Path $templateFile -Value ''
   ```

1. In the PowerShell session of Cloud Shell, use the nano text editor to add the following content to the newly created file:

   ```json
   {
     "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "imageTemplateName": {
         "type": "string"
       },
       "api-version": {
         "type": "string"
       },
       "svclocation": {
         "type": "string"
       }
     },
     "variables": {},
     "resources": [
       {
         "name": "[parameters('imageTemplateName')]",
         "type": "Microsoft.VirtualMachineImages/imageTemplates",
         "apiVersion": "[parameters('api-version')]",
         "location": "[parameters('svclocation')]",
         "dependsOn": [],
         "tags": {
           "imagebuilderTemplate": "win11multi",
           "userIdentity": "enabled"
         },
         "identity": {
           "type": "UserAssigned",
           "userAssignedIdentities": {
             "<imgBuilderId>": {}
           }
         },
         "properties": {
           "buildTimeoutInMinutes": 100,
           "vmProfile": {
             "vmSize": "Standard_DS2_v2",
             "osDiskSizeGB": 127
           },
           "source": {
             "type": "PlatformImage",
             "publisher": "MicrosoftWindowsDesktop",
             "offer": "Windows-11",
             "sku": "win11-21h2-ent",
             "version": "latest"
           },
           "customize": [
             {
               "type": "PowerShell",
               "name": "Install Choco and Vscode",
               "inline": [
                 "Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))",
                 "choco install -y vscode"
               ]
             }
           ],
           "distribute": [
             {
               "type": "SharedImage",
               "galleryImageId": "/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<sharedImageGalName>/images/<imageDefName>",
               "runOutputName": "<runOutputName>",
               "artifactTags": {
                 "source": "azureVmImageBuilder",
                 "baseosimg": "win11multi"
               },
               "replicationRegions": ["<region1>", "<region2>"]
             }
           ]
         }
       }
     ]
   }
   ```

1. Run the following commands to replace placeholders in the template.json with the values specific to your Azure environment:

   ```powershell
   $replRegion2 = 'eastus2'
   $templateFilePath = '.\template.json'
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<subscriptionID>', $subscriptionID | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<rgName>', $imageResourceGroup | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<runOutputName>', $runOutputName | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<imageDefName>', $imageDefName | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<sharedImageGalName>', $computeGallery | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<region1>', $location | Set-Content -Path $templateFilePath
   (Get-Content -Path $templateFilePath -Raw ) -Replace '<region2>', $replRegion2 | Set-Content -Path $templateFilePath
   ((Get-Content -Path $templateFilePath -Raw) -Replace '<imgBuilderId>', $identityNameResourceId) | Set-Content -Path $templateFilePath
   ```

1. Run the following command to submit the template to the Azure Image Builder service (the service processes the submitted template by downloading any dependent artifacts, such as scripts, and storing them in a staging resource group, which name includes the **IT\_** prefix, for building a custom virtual machine image:

   ```powershell
   New-AzResourceGroupDeployment -ResourceGroupName $imageResourceGroup -TemplateFile $templateFilePath -Api-Version "2020-02-14" -imageTemplateName $imageTemplateName -svclocation $location
   ```

1. Run the following command to invoke the image build process:

   ```powershell
   Invoke-AzResourceAction -ResourceName $imageTemplateName -ResourceGroupName $imageResourceGroup -ResourceType Microsoft.VirtualMachineImages/imageTemplates -ApiVersion "2020-02-14" -Action Run -Force
   ```

1. Run the following command to determine the image provisioning state:

   ```powershell
   Get-AzImageBuilderTemplate -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup | Select-Object -Property Name, LastRunStatusRunState, LastRunStatusMessage, ProvisioningState
   ```

   > **Note:** The following output will indicate that the build process has completed successfully:

   ```powershell
   Name                LastRunStatusRunState LastRunStatusMessage ProvisioningState
   ----                --------------------- -------------------- -----------------
   templateWinVSCode01 Succeeded                                  Succeeded
   ```

1. Alternatively, to monitor the build progress, use the following procedure:

   1. In the Azure portal, search for and select **Image templates**.
   1. On the **Image templates** page, select **templateWinVSCode01**.
   1. On the **templateWinVSCode01** page, in the **Essentials** section, note the value of the **Build run state** entry.

   > **Note:** The build process might take about 30 minutes. Do not wait for its completion but instead proceed to the next task of this exercise.

1. Once the build completes, in the Azure portal, search for and select **Azure compute galleries**.
1. On the **Azure compute galleries** page, select **compute_gallery_01**.
1. On the **compute_gallery_01** page, ensure that the **Definitions** tab is selected and, in the list of definitions, select **imageDefDevBoxVSCode**.
1. On the **imageDefDevBoxVSCode** page, select the **Versions** tab and verify that the **1.0.0 (latest version)** entry appears on the list with the **Provisioning State** set to **Succeeded**.
1. Select the **1.0.0 (latest version)** entry.
1. On the **1.0.0 (compute_gallery_01/imageDefDevBoxVSCode/1.0.0)** page, review the VM image version settings.

### Task 4: Create an Azure dev center network connection

In this task, you will configure Azure dev center networking to be used in a scenario that requires private connectivity to resources hosted within an Azure virtual network. Unlike Microsoft hosted network that you leveraged in the first exercise of this lab, virtual network connections also support hybrid scenarios (providing connectivity to on-premises resources) and Microsoft Entra hybrid join of Azure dev boxes (in addition to support for Microsoft Entra join).

1. Within the console session to the lab computer, in the web browser displaying the Azure portal, in the **Search** text box, search for and select **Virtual networks**.
1. On the **Virtual networks** page, select **+ Create**.
1. On the **Basics** tab of the **Create virtual network** page, specify the following settings and then select **Next**:

   | Setting        | Value                                                        |
   | -------------- | ------------------------------------------------------------ |
   | Subscription   | The name of the Azure subscription you are using in this lab |
   | Resource group | **rg-devcenter-01**                                          |
   | Name           | **vnet-01**                                                  |
   | Location       | **(US) East US**                                             |

1. On the **Security** tab of the **Create virtual network** page, review the existing settings without changing their default values and then select **Next**.
1. On the **IP addresses** tab of the **Create virtual network** page, review the existing settings without changing their default values and then select **Review + Create**.
1. On the **Review + Create** tab of the **Create virtual network** page, select **Create**.

   > **Note:** Wait for the virtual network to be created. This should take less than 1 minute.

1. In the Azure portal, in the **Search** text box, search for and select **Network connections**.
1. On the **Network connections** page, select **+ Create**.
1. On the **Basics** tab of the **Create a network connection** page, specify the following settings and then select **Review + Create**:

   | Setting         | Value                                                        |
   | --------------- | ------------------------------------------------------------ |
   | Subscription    | The name of the Azure subscription you are using in this lab |
   | Resource group  | **rg-devcenter-01**                                          |
   | Name            | **network-connection-vnet-01**                               |
   | Virtual network | **vnet-01**                                                  |
   | Subnet          | **default**                                                  |

1. On the **Review + Create** tab of the **Create virtual network** page, select **Create**.

   > **Note:** Wait for the network connection to be created. This might take about 1 minute.

1. In the Azure portal, search for and select **Dev centers** and, on the **Dev centers** page, select **devcenter-01**.
1. On the **devcenter-01** page, in the vertical navigation menu on the left side, expand the **Dev box configuration** section and select **Networking**.
1. On the **devcenter-01 \| Networking** page, select **+ Add**.
1. In the **Add network connection** pane, in in the **Network connection** drop-down list, select **network-connection-vnet-01** and then select **Add**.

   > **Note:** Do not wait for network connection to be created, but instead proceed to the next task. Adding a network connection might take about 1 minute,

### Task 5: Adding image definitions to an Azure dev center project

In this task, you will add image definitions to an Azure dev center project you created in the first exercise of this lab. Image definitions combine an Azure Marketplace or a custom image with configurable tasks that define additional modifications to be applied to the underlying image. An image definition can be used to build a new image (containing all changes, including those applied by tasks) or to create dev box pools directly. Creating a reusable image minimizes time required for dev box provisioning.

To configure imaging for Microsoft Dev Box team customizations, project-level catalogs must be enabled (which you already completed in the first exercise of this lab). In this task, you will configure catalog sync settings for the project. This will involve attaching a catalog that contains image definition files.

1. Within the console session to the lab computer, in the web browser displaying the Azure portal, in the **Search** text box, search for and select **Dev centers**.
1. On the **Dev centers** page, select **devcenter-01**.
1. On the **devcenter-01** page, in the vertical navigation menu on the left side, expand the **Manage** section and select **Projects**.
1. On the **devcenter-01 \| Projects** page, in the list of projects, select **devcenter-project-01**.
1. On the **devcenter-project-01** page, in the vertical navigation menu on the left side, expand the **Settings** section and select **Catalogs**.
1. On the **devcenter-project-01 \| Catalogs** page, select **+ Add**.
1. In the **Add catalog** pane, in the **Name** text box, enter **image-definitions-01**, in the **Catalog source** section, select **GitHub**, in the **Authentication type**, select **GitHub app**, leave the checkbox **Automatically sync this catalog** checkbox enabled, and then select **Sign in with GitHub**.
1. If prompted, in the **Sign in with GitHub** window, enter the GitHub credentials provided in the lab environment and select **Sign in**.

   > **Note:** These GitHub credentials provide you with access to a GitHub repo created as a fork of https://github.com/dhruvchand/contoso-co-eShop

1. If prompted, in the **Authorize Microsoft DevCenter** window, select **Authorize Microsoft DevCenter**.
1. Back in the **Add catalog** pane, in the **Repo** drop-down list, select **contoso-co-eShop**, in the **Branch** drop-down list, accept the **Default branch** entry, in the **Folder path**, enter **.devcenter/catalog/image-definitions** and then select **Add**.
1. Back on the **devcenter-project-01 \| Catalogs** page, verify that the sync completes successfully by monitoring the entry in the **Status** column.
1. Select the **Sync successful** link in the **Status** column, review the resulting notification pane, verify 3 items were added to the catalog, and close the pane by selecting the **x** symbol in the upper right corner.
1. Back on the **devcenter-project-01 \| Catalogs** page, select **image-definitions-01** and verify that it contains three entries named **ContosoBaseImageDefinition**, **backend-eng**, and **frontend-eng**.

   > **Note:** For the purpose of this lab, you will test the functionality of the **frontend-eng** image definition, which has the following content:

   ```yaml
   $schema: "1.0"
   name: "frontend-eng"
   # Using the "Windows 11 Enterprise 24H2" image as the base
   image: microsoftwindowsdesktop_windows-ent-cpc_win11-24H2-ent-cpc
   description: "This definition is for the eShop frontend engineering environment"

   tasks:
     - name: ~/winget
       description: Install Visual Studio Code
       parameters:
         package: Microsoft.VisualStudioCode
         runAsUser: true
   ```

1. In the Azure portal, navigate back to the **devcenter-project-01** page, in the vertical navigation menu on the left side, expand the **Manage** section, select **Image definitions**, and verify that the page displays the same 3 image definitions you identified earlier in this task.

### Task 6: Create a customized dev box pool

In this task, you will use the newly provisioned image definitions to create a dev box pool. The pool will also utilize the network connection you set up earlier in this exercise.

1. In the Azure portal displaying the **devcenter-project-01 \| Image definitions** page, in the vertical navigation menu on the left side, in the **Manage** section, select **Dev box pools**.
1. On the **devcenter-project-01 \| Dev box pools** page, select **+ Create**.
1. On the **Basics** tab of the **Create a dev box pool** page, specify the following settings and then select **Create**:

   | Setting                                                                                                           | Value                                                 |
   | ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
   | Name                                                                                                              | **devcenter-project-01-devbox-pool-02**               |
   | Definition                                                                                                        | **frontend-eng**                                      |
   | Compute                                                                                                           | **8 vCPU, 32 GB RAM**                                 |
   | Storage                                                                                                           | **256 GB SSD**                                        |
   | Network connection                                                                                                | **Deploy to a network connection in my organization** |
   | Network connection name                                                                                           | **network-connection-vnet-01**                        |
   | Enable single sign-on                                                                                             | Enabled                                               |
   | Dev box Creator Privileges                                                                                        | **Local Administrator**                               |
   | Enable auto-stop on schedule                                                                                      | Enabled                                               |
   | Stop time                                                                                                         | **07:00 PM**                                          |
   | Time zone                                                                                                         | Your current time zone                                |
   | Enable hibernate on disconnect                                                                                    | Enabled                                               |
   | Grace period in minutes                                                                                           | **60**                                                |
   | I confirm that my organization has Azure Hybrid Benefits licenses, which will apply to all dev boxes in this pool | Enabled                                               |

   > **Note:** Wait for the dev box pool to be created. This might take about 2 minutes.

### Task 7: Evaluate a customized dev box

In this task, you will evaluate the functionality of a customized dev box by using a Microsoft Entra developer user account.

> **Note:** Considering extra time required to complete this task, its completion is optional.

1. Within the console session to the lab computer, start a web browser incognito/in-private and navigate to the Microsoft Dev Box developer portal at `https://aka.ms/devbox-portal`.
1. When prompted to sign in, provide the credentials of the **devuser01** user account.
1. On the **Welcome, devuser01** page of the Microsoft Dev Box developer portal, select **+ New dev box**.
1. In the **Add a dev box** pane, in the **Name** text box, enter **devuser01box02**.
1. In the **Dev box pool** drop-down list, select **devcenter-project-01-devbox-pool-02**.
1. Review other information presented in the **Add a dev box** pane, including the project name, dev box pool specifications, hibernation support status, and the scheduled shutdown timing. In addition, note the option to apply customizations and the notification that dev box creation might take up to 65 minutes.
1. In the **Add a dev box** pane, select **Create**.
1. Once the dev box is fully provisioned and running, connect to it by selecting the option to **Connect via app**.
1. In the pop-up window titled **This site is trying to open Microsoft Remote Connection Center**, select **Open**. This will automatically initiate a Remote Desktop session to the dev box.
1. When prompted for credentials, authenticate by providing the user name and the password of the **devuser01** account.
1. Within the Remote Desktop session to the dev box, verify that its configuration includes an installation of Visual Studio Code.

   > **Note:** You can also validate the outcome of customization by by selecting the ellipsis symbol in the **Your dev box** interface and then selecting **Customizations** from the cascading menu.
