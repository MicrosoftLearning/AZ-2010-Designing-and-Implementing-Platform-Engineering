---
lab:
  title: "Implementing Real-Time Monitoring with Azure Monitor"
  module: "Observability and Continuous Improvement"
---

# Lab 02: Implementing Real-Time Monitoring with Azure Monitor

## Estimated timing: 30 minutes

## Prerequisites

## Objectives

- Deploy a sample web application in Azure.
- Enable and configure Azure Monitor and Application Insights.
- Create custom dashboards to visualize application metrics.
- Set up alerts to notify stakeholders of performance anomalies.
- Analyze performance data for potential improvements.

## Exercise 1: Deploy a sample web application

As a platform engineer, you need to ensure the applications running on your platform are observable and continuously monitored. In this lab, you will deploy a sample web application to Azure, configure Azure Monitor and Application Insights, set up custom dashboards, and create alerts to track application performance.

### Task 1: Create an Azure Web App

1. Open the Azure portal.
1. In the Search bar, type App Services and select it.
1. Click + Create.
1. In the Basics tab:
   - Subscription: Select your Azure subscription.
   - Resource Group: Click Create new, enter MonitoringLab-RG, and click OK.
   - Name: Enter a unique name, such as monitoringlab-webapp.
   - Publish: Select Code.
   - Runtime stack: Choose .NET 8 (LTS).
   - Region: Select a region close to you.
1. Click Review + Create, then click Create.
1. Wait for deployment to complete, then click Go to resource.

### Task 2: Enable Application Insights

1. In the Web App resource, scroll to the Monitoring section.
1. Click Application Insights.
1. Click Enable.
1. Select Create new resource and keep the default settings.
1. Click Apply and wait for the configuration to complete.

## Exercise 2: Configure Azure Monitor and dashboards

### Task 1: Access Azure Monitor

### Task 2: Add key metrics to the dashboard

## Exercise 3: Create alerts

### Task 1: Define alert conditions

### Task 2: Define an action group

## Exercise 4: Analyze performance data
