# Azure DevOps Powershell to AzureSQL

This repo shows how you can run a sql script against a database using PowerShell in one of three ways.

1. Azure SQL Database Deployment task - shields the user from all complexity of making the connection to Azure and getting an access token.
2. Azure Powershell task - shields the user from the complexity of making the connection to Azure, but not from getting an access token.
3. Powershell task - requires the user to manage the Service Principal credentials, establish the connection to Azure, and get an access token.

All options leverage the SQLServer powershell module.  However, the Azure Powershell and Powershell tasks require a [self hosted pool](https://learn.microsoft.com/azure/devops/pipelines/agents/v2-windows?view=azure-devops) to run the pipeline on since the built in agents are unable to install PowerShell modules.

1. Create an active directory account to use to connect to the database

    Option 1 - Use a DevOps Service Connection (recommended!)

        - In Azure DevOps, click on 'Project Settings'.
        - Select 'Service Connections'.
        - Select 'New service connection'.
        - Select 'Azure Resource Manager' and click 'Next'
        - Select 'Service principal (automatic) and click 'Next'.
        - Enter your subscription, resource group, and service connection name and click 'Save'.
        - Open up your new connection and select 'Manage Service Principal'.
        - Copy the 'Display name' of the Service Principal to notepad.

    Option 2 - Manually create a Service Principal

        - Open an Azure Cloud Shell bash session.
        - Create a service principal

            `az ad sp create-for-rbac -n <YourServicePrincipalName>`

        - Save the output to notepad.

2. Sign into the database using your AD credentials.

3. Create a login on the master database.

    `CREATE LOGIN [<YourServicePrincipalName>] FROM EXTERNAL PROVIDER;`

4. Create a user on you application database.

    `CREATE USER [<YourServicePrincipalName>] FROM LOGIN [<YourServicePrincipalName>]`

5. Create a Pipeline

    - Create a new yaml pipeline in Azure Dev Ops.
    - Copy and paste the contents of [pipeline.yml](/pipeline.yml)
    - If you're using the SQL Deployment (SqlAzureDacpacDeployment@1) task, delete the PowerShell tasks and enter your Service Connection, Database Server Name, and Database Name.
    - If you're using a Service Connection (AzurePowerShell@5) task, delete the SqlAzureDacpacDeployment@1 andPowerShell@2 tasks and enter your Service Connection, Database Server Name, and Database Name.
    - If you're using a Service Principal (PowerShell@2), delete the SqlAzureDacpacDeployment@1 and AzurePowerShell@5 task and enter in your Service Principal detals, Database Server Name, and Database Name. Note: This code is for demonstration purposes!  You should store the Service Principal credentials in Azure Key Vault!!
