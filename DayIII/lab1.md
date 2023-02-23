# LAB 1 : Create CD Repository pipeline on GitHub

<div align="center"><img src="../img/image-20221206-071637.png" width="100%"></div>



Learn how to create repository CD on GitHub and working to preparing to deploy pipeline to Azure.

After completing this lab, you'll be able to:

- Describe or explain how to design and implement CD pipeline simple.
- Preparing to implement CD pipeline, simple to deploy Azure App service (Web app).
- Connecting between GitHub Self-hosted runner and GitHub repository pipeline

In this lab we use sample application that is named Tutorial API Backend and Tutorial Frontend, we focus on Backend application to deploy on Azure App Service (Web app).

## Prerequisites

- <b>Workspace that required Software and Tools</b> 
    - Git and GitHub Account
    - Text Editor (Required Visual Studio Code, or Visual Studio) [Visual Studio Code - Code Editing. Redefined](https://code.visualstudio.com/)
    - AZ CLI ([How to install the Azure CLI | Microsoft Learn](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli))
- <b>Infrastructures or Resources on Azure</b>
    - Azure App service (Webapp support deploy code and dotnet6) 
    - Azure App service plan (Windows or Linux)
    - Azure Cosmos DB for MongoDB API (Step for Initialize cosmos DB)
    - Azure Key Vault (if any)
    - Azure Application Insights (if any)
- Required Lab5 CI

## 1. Create new Repository on GitHub
On GitHub `github.com/<username>` create new Repository named `<username>-pipeline`

on the top right and choose New repository tab.

<div align="center"><img src="../img/image-20221206-065945.png" width="100%"></div>

On this page Create new repository selects in below
- Owner: Username
- Repository name: <username>-pipeline
- Description: Optional
- Public
    - Initialize this repository with
    - Add a README.md file
- Initial branch, select main to default.

<div align="center"><img src="../img/image-20221206-070159.png" width="100%"></div>

<div align="center"><img src="../img/image-20221206-070309.png" width="100%"></div>


---

## 2. Create Virtual Machine

Go to the Azure portal [Microsoft Azure](http://portal.azure.com/)

Inside resource group named `rg-<name>-az-asse-sbx-001`

<div align="center"><img src="../img/image-20230207-081555.png" width="100%"></div>

Select Compute or you can find in Search the Marketplace `“Ubuntu Sever xx.xx“`

<div align="center"><img src="../img/image-20230207-081627.png" width="100%"></div>

Click <b>Create</b>

<div align="center"><img src="../img/image-20230207-081817.png" width="100%"></div>


In the page, Enter information in below

- Subscription: `subscription-name`
- Resource group: `rg-<name>-az-asse-sbx-001`
- Instance details
    - Virtual machine name: `vm-<name>SelfHost-az-asse-sbx-001`
    - Region: `<region>`
    - Run with Azure Spot discount: `true`
    - Size: `Standard_D2s_v3 - 2 vcpu, 8 GiB`
    - Authentication type: `Default`

<div align="center"><img src="../img/image-20230207-084714.png" width="100%"></div>

<div align="center"><img src="../img/image-20230207-082945.png" width="100%"></div>

<div align="center"><img src="../img/image-20230207-083008.png" width="100%"></div>

On Networking page select

Delete public IP and NIC when VM is deleted: `true`

<div align="center"><img src="../img/image-20230207-083222.png" width="100%"></div>





