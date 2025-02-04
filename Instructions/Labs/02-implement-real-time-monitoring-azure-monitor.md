---
lab:
  title: "Implementing Real-Time Monitoring with Azure Monitor"
  module: "Observability and Continuous Improvement"
---

# Lab 02: Implementing Real-Time Monitoring with Azure Monitor

## Estimated timing: 20 minutes

## Prerequisites

- Azure subscription - If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/).
- Basic knowledge of monitoring and observability concepts.
- Basic understanding of Azure services.

## Objectives

- Create a sample web app.
- Enable and configure Azure Monitor and Application Insights.
- Create custom dashboards to visualize application metrics.
- Set up alerts to notify stakeholders of performance anomalies.
- Analyze performance data for potential improvements.

## Exercise 1: Create a sample web application

As a platform engineer, you need to ensure the applications running on your platform are observable and continuously monitored. In this lab, you will create a sample web application to Azure, configure Azure Monitor and Application Insights, set up custom dashboards, and create alerts to track application performance.

### Task 1: Create an Azure Web App

1. Open the Azure portal.
1. In the Search bar, type App Services and select it.
1. Click + Create and select Web App.
1. In the Basics tab:
   - Subscription: Select your Azure subscription.
   - Resource Group: Click Create new, enter monitoringlab-rg, and click OK.
   - Name: Enter a unique name, such as monitoringlab-webapp.
   - Publish: Select Code.
   - Runtime stack: Choose .NET 8 (LTS).
   - Region: Select a region close to you.
1. Click Review + create, then click Create.
1. Wait for deployment to complete, then click "Go to resource".
1. In the Overview tab, click the URL to verify the web app is running.

### Task 2: Verify Application Insights

1. In the Web App resource, scroll to the Monitoring section.
1. Click Application Insights.
1. Application Insights is already enabled for this web app. Click the link to open the Application Insights resource.
1. In the Application Insights resource, click in the Application Dashboard to view the performance data the default dashboard provided.

## Exercise 2: Configure Azure Monitor and dashboards

### Task 1: Access Azure Monitor

1. In the Azure portal, search for Monitor and click on it.
1. Click Metrics in the left panel.
1. In the Scope section, select the web app under the subscription and resource group where you deployed the web app.
1. Click Apply and observe the metrics available for the web app.

### Task 2: Add key metrics to the dashboard

1. In the Scope section, select your Web App (monitoringlab-webapp).
1. Under Metric, choose Response Time.
1. Set the Aggregation to Average and click + Add metric.
1. Repeat the process for additional metrics:
   - CPU Time (Count)
   - Requests (Average)
1. Click to dashboard and pin to dashboard.
1. Select type Shared and select the subscription and the monitoringlab-webapp Dashboard.
1. Click Pin to dashboard.
1. Click the dashboard icon in the left panel to view the dashboard.
1. Verify the metrics are displayed on the dashboard and are updating in real-time.

## Exercise 3: Create alerts

### Task 1: Define alert conditions and actions

1. In Azure Monitor, click Alerts.
1. Click + Create and select Alert rule.
1. Under Scope, select your Web App (monitoringlab-webapp) and click Apply.
1. Under Condition, click in the Signal name field and select Response Time.
1. Configure the alert rule:
   - Threshold type: Dynamic
   - Aggregation type: Average
   - Value is: Greater or Less than
   - Threshold Sensitivity: High
   - When to evaluate: Check every 1 minutes and look back at 5 minutes.
1. Under Actions, select Use quick actions.
1. Enter:
   - Action group name: WebAppMonitoringAlerts
   - Display name: WebAlert
   - Email: Enter your email address.
1. Click Save.
1. Click Next: Details.
1. Enter a Name `WebAppResponseTimeAlert` and select a Severity level Verbose.
1. Click Review + create and then Create.

   > **Note:** Your alert rule is now created and will trigger an email notification when the response time exceeds the threshold. You can force the alert to trigger by sending a large number of requests to the web app. For example, you can use Azure Load testing or a tool like Apache JMeter.

1. Go back to the Azure Monitor > Alerts.
1. Click on Alert rules and you should see the alert rule you created.

## Exercise 4: Analyze performance data

### Task 1: Review Collected Metrics

1. Go to Application Insights in the Azure portal.
1. Click on the Application Dashboard.
1. Click Performance tile to analyze server response times and load. You can also view the number of requests and failed requests.
1. Click on Analyze with Workbooks and select Performance Counter Analysis.
1. Click Workbooks.
1. Select the workbook Performance Analysis under Performance.
1. You can see the performance data for the web app.

> **Note:** You can customize the workbook to include additional metrics and filters.
