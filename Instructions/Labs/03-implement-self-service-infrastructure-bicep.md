---
lab:
  title: "Implementing Self-Service Infrastructure with Bicep"
  module: "Strategic Platform Road Mapping"
---

# Lab 03: Implementing Self-Service Infrastructure with Bicep

## Estimated timing: 20 minutes

## Prerequisites

- Azure subscription - If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/).
- Basic knowledge of Azure services and Azure CLI.
- Visual Studio Code installed with the Bicep extension.
- Azure CLI installed and configured on your local machine.
- Familiarity with Infrastructure as Code (IaC) concepts.

## Objectives

- Set up Bicep in your environment.
- Create a Bicep template to define Azure resources.
- Deploy an Azure App Service with a SQL Database backend.
- Enforce governance using tags and policies.
- Implement automated scaling with Bicep.

## Exercise 1: Create a Bicep template to deploy Azure resources

In a platform engineering environment, developers need a way to provision infrastructure consistently and efficiently. This lab will guide you through using Bicep, an Infrastructure as Code (IaC) tool, to deploy and manage Azure resources in a self-service manner. You will create a Bicep template to provision a resource group, an Azure App Service, an Azure SQL Database, and a Storage Account while enforcing governance with tagging and policies.

### Task 1: Install Bicep CLI

1. Open your local terminal.
1. To verify if Bicep is installed, run:

   ```bash
   az bicep version
   ```

   If Bicep is not installed, install it with:

   ```bash
   az bicep install
   ```

1. Confirm the installation by running:

   ```bash
   az bicep version
   ```

   You should see output like Bicep CLI version X.Y.Z.

   > **NOTE:** Ensure you have the [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) installed.

### Task 2: Create a Bicep Template

1. In your local terminal, create a new file:

   ```bash
   code main.bicep
   ```

1. Copy and paste the following Bicep code into the file:

   ```bicep
   param location string = 'eastus2'
   param appServicePlanName string = 'bicepAppPlan'
   param webAppName string = 'bicep-webapp'
   param storageAccountName string = 'biceplabstorage'
   param sqlServerName string = 'bicep-sqlserver'
   param sqlDatabaseName string = 'bicepdb'
   param sqlAdminUser string
   @secure()
   param sqlAdminPassword string

   targetScope = 'resourceGroup'

   resource storage 'Microsoft.Storage/storageAccounts@2021-09-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'StorageV2'
   }

   resource appPlan 'Microsoft.Web/serverfarms@2021-02-01' = {
     name: appServicePlanName
     location: location
     sku: {
       name: 'F1'
       tier: 'Free'
     }
   }

   resource webApp 'Microsoft.Web/sites@2021-02-01' = {
     name: webAppName
     location: location
     properties: {
       serverFarmId: appPlan.id
     }
   }

   resource sqlServer 'Microsoft.Sql/servers@2022-05-01-preview' = {
     name: sqlServerName
     location: location
     properties: {
       administratorLogin: sqlAdminUser
       administratorLoginPassword: sqlAdminPassword
       version: '12.0'
     }
   }

   resource sqlDb 'Microsoft.Sql/servers/databases@2022-05-01-preview' = {
     name: sqlDatabaseName
     location: location
     parent: sqlServer
     properties: {
       collation: 'SQL_Latin1_General_CP1_CI_AS'
       maxSizeBytes: 2147483648
     }
   }
   ```

   > **NOTE:** You can install the Bicep extension in Visual Studio Code to get syntax highlighting and IntelliSense for Bicep files.

   > **IMPORTANT:** Ensure storageAccountName, webAppName, and sqlServerName are globally unique. If deployment fails due to name conflicts, modify the names accordingly.

   > **NOTE:** If you encounter an error related to the region, try deploying to a different region by changing the `location` parameter value.

1. Save and close the file.

### Task 3: Deploy the template with Azure CLI

1. Run the following command to create the resource group:

   ```bash
   az group create --name BicepLab-RG --location centralus

   ```

   > **NOTE:** You may need to log in to your Azure account if you are not already authenticated. For detailed instructions, see [Authenticate to Azure using Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli).

2. Deploy the template within the resource group:

   ```bash
   az deployment group create --resource-group BicepLab-RG --template-file main.bicep --parameters sqlAdminUser='azureuser' sqlAdminPassword='YourSecurePassword123!'

   ```

   > **NOTE:** Replace YourSecurePassword123! with a strong password and azureuser with a valid username.

3. Wait for the deployment to complete. You should see a confirmation message.
4. In the Azure portal, navigate to Resource Groups.
5. Locate BicepLab-RG and click on it.
6. Verify that the resources were created successfully.

You have successfully deployed an Azure App Service with a SQL Database backend using a Bicep template. You can now manage your infrastructure as code and deploy resources consistently and efficiently. The next step would be to integrate this template into your CI/CD pipeline for automated deployments.

## Exercise 2: Enforce governance with tags and policies

In a self-service infrastructure environment, it is essential to enforce governance to ensure compliance and cost control. Tags and policies are two key mechanisms to achieve this.

### Task 1: Add tags to the resources

1. Run the following command to add a tag to the resource group:

   ```bash
   az group update --name BicepLab-RG --set tags.Owner='PlatformEngineering'
   ```

1. Verify that the tag was added by running:

   ```bash
   az group show --name BicepLab-RG --query tags
   ```

### Task 2: Create a policy to enforce tagging

1. Create a JSON policy file:

   ```bash
   code tagging-policy.json
   ```

1. Copy and paste the following policy definition into the file:

   ```json
   {
     "if": {
       "not": {
         "field": "tags[Owner]",
         "exists": "true"
       }
     },
     "then": {
       "effect": "deny"
     }
   }
   ```

1. Create the policy definition using the JSON file:

   ```bash
    az policy definition create --name 'tagging-policy' --display-name 'Enforce tagging' --rules @tagging-policy.json --mode All
   ```

1. Assign the policy to the resource group:

   ```bash
   az policy assignment create --name 'tagging-policy-assignment' --display-name 'Enforce tagging' --policy "/subscriptions/<subscription-id>/providers/Microsoft.Authorization/policyDefinitions/tagging-policy" --scope "/subscriptions/<subscription-id>/resourceGroups/BicepLab-RG"

   ```

   > **IMPORTANT:** Replace both <subscription-id> with your Azure subscription ID.

1. Verify that the policy was assigned by running:

   ```bash
   az policy assignment list --output table
   ```

### Task 3: Test the policy enforcement

1. Run the following command to remove the tag from the resource group:

   ```bash
   az tag update --resource-id /subscriptions/<subscription-id>/resourceGroups/BicepLab-RG --operation delete --tags Owner
   ```

   > **IMPORTANT:** Replace <subscription-id> with your Azure subscription ID.

1. You should see an error message indicating that the operation was denied due to the policy enforcement. The tag should not be removed. This confirms that the policy is working as expected. To resolve the error, temporarily remove the policy assignment and re-run the command.

   ```bash
   az policy exemption create --name 'tagging-policy-exemption' --policy-assignment "/subscriptions/<subscription-id>/resourceGroups/BicepLab-RG/providers/Microsoft.Authorization/policyAssignments/tagging-policy-assignment" --scope "/subscriptions/<subscription-id>/resourceGroups/BicepLab-RG" --display-name "Temporary exemption to remove tag" --exemption-category Waiver
   ```

   > **IMPORTANT:** Replace both <subscription-id> with your Azure subscription ID.

You have successfully enforced governance using tags and policies. Developers will now be required to tag resources with the Owner tag, ensuring proper ownership and accountability. You can create additional policies to enforce other governance requirements, such as resource naming conventions, resource types, and resource locations.

## Exercise 3: Implement automated scaling with Bicep

In a platform engineering environment, ensuring that applications can scale efficiently is a key requirement. Automated scaling improves performance, optimizes resource utilization, and minimizes costs. In this exercise, you will implement autoscaling for an Azure App Service using Bicep.

### Task 1: Define an autoscaling policy in Bicep

1. Open the main.bicep file and locate the existing App Service Plan section:

   ```bicep
   resource appPlan 'Microsoft.Web/serverfarms@2021-02-01' = {
     name: appServicePlanName
     location: location
     sku: {
       name: 'F1'
       tier: 'Free'
     }
   }
   ```

1. Modify the appPlan resource to use the S1 SKU (which supports autoscaling):

   ```bicep
   resource appPlan 'Microsoft.Web/serverfarms@2021-02-01' = {
     name: appServicePlanName
     location: location
     sku: {
       name: 'S1'
       tier: 'Standard'
     }
     properties: {
       perSiteScaling: false
       maximumElasticWorkerCount: 10
     }
   }
   ```

1. Immediately after this resource, add the autoscaleSetting configuration:

   ```bicep
   resource autoscaleSetting 'Microsoft.Insights/autoscaleSettings@2024-01-01-preview' = {
   name: 'autoscale-rule'
   location: location
   properties: {
    profiles: [
      {
        name: 'defaultProfile'
        capacity: {
          minimum: '1'
          maximum: '5'
          default: '1'
        }
        rules: [
          {
            metricTrigger: {
              metricName: 'CpuPercentage'
              metricResourceUri: resourceId('Microsoft.Web/serverfarms', appServicePlanName)
              operator: 'GreaterThan'
              threshold: 75
              timeAggregation: 'Average'
              statistic: 'Average'
              timeWindow: 'PT5M'
              timeGrain: 'PT1M'
            }
            scaleAction: {
              direction: 'Increase'
              type: 'ChangeCount'
              value: '1'
              cooldown: 'PT1M'
            }
          }
        ]
      }
    ]
   }
   }
   ```

1. Save the file.
1. Validate the updated Bicep file:

   ```bash
   az bicep build --file main.bicep
   ```

1. Deploy the updated template:

   ```bash
   az deployment group create --resource-group BicepLab-RG --template-file main.bicep --parameters sqlAdminUser='azureuser' sqlAdminPassword='YourSecurePassword123!'

   ```

1. Verify that the autoscaling policy was applied to the App Service Plan. Go to the Azure portal, navigate to the App Service Plan, select the Scale out (App Service Plan) blade, and verify that the autoscale rule is configured.

   > **NOTE:** If you want to test the autoscaling behavior, you can simulate high CPU usage on the App Service by running a load test or generating traffic. The App Service should automatically scale out based on the defined rule.

You have successfully implemented automated scaling for an Azure App Service using Bicep. This will ensure that your applications can handle increased traffic and demand efficiently, improving performance and reducing costs.
