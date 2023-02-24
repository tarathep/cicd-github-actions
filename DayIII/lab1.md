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

<div align="center"><img src="../img/image-20221206-065945.png" width="90%"></div>

On this page Create new repository selects in below
- Owner: Username
- Repository name: <username>-pipeline
- Description: Optional
- Public
    - Initialize this repository with
    - Add a README.md file
- Initial branch, select main to default.

<div align="center"><img src="../img/image-20221206-070159.png" width="90%"></div>

<div align="center"><img src="../img/image-20221206-070309.png" width="90%"></div>


---

## 2. Create Virtual Machine

Go to the Azure portal [Microsoft Azure](http://portal.azure.com/)

Inside resource group named `rg-<name>-az-asse-sbx-001`

<div align="center"><img src="../img/image-20230207-081555.png" width="90%"></div>

Select Compute or you can find in Search the Marketplace `“Ubuntu Sever xx.xx“`

<div align="center"><img src="../img/image-20230207-081627.png" width="90%"></div>

Click <b>Create</b>

<div align="center"><img src="../img/image-20230207-081817.png" width="90%"></div>


In the page, Enter information in below

- Subscription: `subscription-name`
- Resource group: `rg-<name>-az-asse-sbx-001`
- Instance details
    - Virtual machine name: `vm-<name>SelfHost-az-asse-sbx-001`
    - Region: `<region>`
    - Run with Azure Spot discount: `true`
    - Size: `Standard_D2s_v3 - 2 vcpu, 8 GiB`
    - Authentication type: `Default`

<div align="center"><img src="../img/image-20230207-084714.png" width="90%"></div>

<div align="center"><img src="../img/image-20230207-082945.png" width="90%"></div>

<div align="center"><img src="../img/image-20230207-083008.png" width="90%"></div>

On Networking page select

Delete public IP and NIC when VM is deleted: `true`

<div align="center"><img src="../img/image-20230207-083222.png" width="90%"></div>

<div align="center"><img src="../img/image-20230207-083413.png" width="90%"></div>

Review + create

<div align="center"><img src="../img/image-20230207-083525.png" width="90%"></div>

and then download Generate new key pair, you can see the file key.pem to use login and access to VM

<div align="center"><img src="../img/image-20230207-083556.png" width="90%"></div>

<div align="center"><img src="../img/image-20230207-083738.png" width="90%"></div>

When deploy was done, <b>Go to resource</b>

<div align="center"><img src="../img/image-20230207-085011.png" width="90%"></div>

---

## 3. Install GitHub Action Runner with Script

Go to the VM (<i>vm-nameSelfHost-az-asse-sbx-001</i>)


Click <b>Connect</b> button, select via SSH.

<div align="center"><img src="../img/image-20230207-085148.png" width="90%"></div>

Connect via SSH with client.

<div align="center"><img src="../img/image-20230207-085256.png" width="90%"></div>

Open PowerShell, Terminal or CMD and enter command below.

```bash
ssh -i <private key path> azureuser@X.xxx.xxx.xx
```

<div align="center"><img src="../img/image-20230207-085649.png" width="90%"></div>

<div align="center"><img src="../img/image-20230207-085732.png" width="90%"></div>

Checkout or Download Script from [Github SelfHosted Setting up 1.1.2](https://gist.github.com/tarathep/16764aa0d90440fd26ddda3bb5a2aa10) to Install GitHub Actions Runner

```bash
git clone https://gist.github.com/tarathep/16764aa0d90440fd26ddda3bb5a2aa10
```

change directory to inside directory you can see script named <b>setup-selfhost-1.x.x.sh</b>

Get Personal Token for login [GitHub Packages: Your packages, at home with their code](http://ghcr.io/) and connect to GitHub repos pipeline CD

Go to your profile on top right corner <i>=> Settings => Developer settings</i>


<div align="center">
<img src="../img/image-20230207-090715.png" width="45%" style="margin:0px 0px 0px 10px">
<img src="../img/image-20230207-090833.png" width="45%" style="margin:0px 0px 0px 10px">
</div>

On left menu, <i>select Token (classic) > Generate new token > Generate new token (classic)</i>

<div align="center"><img src="../img/image-20230207-091010.png" width="90%"></div>

<b>Enter in Note: WORKFLOW_TOKEN_LAB</b>

Select scopes.
- repo
- workflow
- write: package.
- delete: package


<div align="center"><img src="../img/image-20230207-091247.png" width="90%"></div>

Copy secret token

<div align="center"><img src="../img/image-20230207-091627.png" width="90%"></div>

<b>Config Environment Variables into VM Self-hosted</b>

```bash
export GITHUB_PACKAGE_URL=ghcr.io
export GITHUB_PACKAGE_USERNAME=<githubusername>
export GITHUB_PACKAGE_TOKEN=ghp_xxxxxxxxxxxxxxxxx
export GITHUB_PACKAGE_IMAGE=ghcr.io/corp-ais/az-selfhost-runner:latest
export GITHUB_RUNNER_REPO_URL=https://github.com/<username>/<name>-pipeline
export GITHUB_RUNNER_ACCESS_TOKEN=ghp_xxxxxxxxxxxxxxxxx
export GITHUB_RUNNER_NAME_PREFIX=tutorial-be-dev
export GITHUB_RUNNER_LABELS=vm-<name>SelfHost-az-asse-sbx-001,tutorial-be-dev
export GITHUB_RUNNER_NAME_PREFIX=tutorial-be-dev
export GITHUB_RUNNER_NUMBER=1
export AZURERM_VM_ADMIN_USER=azureuser
```

<div align="center"><img src="../img/image-20230207-094001.png" width="90%"></div>

Change Permission and Run Script

```bash
chmod 777 setup-sefhost-1.1.2.sh
sudo -E ./setup-sefhost-1.1.2.sh
```

<div align="center"><img src="../img/image-20230207-100923.png" width="90%"></div>

<div align="center"><img src="../img/image-20230207-101040.png" width="90%"></div>

Done :D







