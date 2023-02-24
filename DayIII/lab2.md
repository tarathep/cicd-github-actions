# LAB 2 : Implement workflows deploy to App service on DEV Environment

<div align="center"><img src="../img/image-20221206-074126.png" width="90%"></div>

Learn how continuous deployment or delivery application to Azure that working on GitHub Actions and implementation basic concepts.

After completing this lab, you'll be able to:

- Explain and Implement CD workflows with GitHub Actions in fundamental.
- Explain Branching strategy working with each environment.
- Automation workflows and how to use Actions.
- Investigate and solvable working on pipeline.

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
- Required Lab1

## Checking Resources Ready

On the Azure Spoke checking list below
- `app-<username>-az-<region>-dev-001`
- `app-<username>-az-<region>-sit-001`
- `id-<username>SelfHost-az-<region>-sbx-001`
- `mongo-<username>-az-<region>-sbx-001`
- `vm-<username>SelfHost-az-<region>-sbx-001`

<div align="center"><img src="../img/image-20221206-073804.png" width="90%"></div>

## Initialize GitHub workflow.

Checkout the source code from GitHub `github.com/<username>/<username>-pipeline`.

Open the terminal following command below.

```bash
git clone https://github.com/<username>/<username>-pipeline.git
```

Open the project with text editor (Visual Studio Code) and create new folder named `github/workflows `in root directory of project (git).

```bash
mkdir .github/workflows
```
<div align="center"><img src="../img/image-20221206-073948.png" width="90%"></div>

GitHub workflow is working on inside `.github/workflows` that contains GitHub workflows files that extension named `.yaml`

## Create DEV - Tutorial BE Deploy workflows

On Create `DEV - Tutorial BE Deploy` workflows is working on workflows CI Dev dispatch and automate deploy to Azure Webapp.

<div align="center"><img src="../img/image-20221206-074126.png" width="90%"></div>

Create the new file named `dev-tutorial-be-deploy.yml` inside `.github/workflows` this on the Workflows Dev which contains 2 workflows: Build and DEV Deploy.

<div align="center"><img src="../img/image-20221206-074147.png" width="90%"></div>

### Name

The first name starts with declare name of workflows

```yaml
name: DEV - Tutorial BE Deploy
```

### On (Events that trigger workflows)

Enter events when do you want to execute or trigger the workflows

you can see more of event type at [Events that trigger workflows - GitHub Docs](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

```yaml
on:
  repository_dispatch:
    types: [<username>-tutorial-be-cd-dev]
  workflow_dispatch:
    inputs:
      ref:
        description: "Repository branch or tag"
        required: true
        default: "develop"

```

### Env

the environment global to declaration and variables

```yaml
env:
  REPOSITORY: "<username>/<username>-tutorial-backend"
  GITREF: ${{ github.event_name == 'repository_dispatch' && 'develop' || github.event.inputs.ref }}
  ARTIFACT_NAME: "artifact-tutorial-be-0.0.1-snapshot"
  APP_RESOURCE_NAME: app-<username>-az-<region>-dev-001

```

### Jobs

Groups together all the jobs that run in the `DEV - Tutorial BE Deploy` workflow.

```yaml
jobs:
...

```

### Build

The build artifact to contains in the job

```yaml
  build-app-service:
    name: Build
    runs-on: ubuntu-latest
    environment:
      name: dev
```

#### Steps

```yaml
    steps:
    - name: "Checkout project"
      uses: actions/checkout@v2
      with:
        repository: ${{ env.REPOSITORY }}
        ref: ${{ env.GITREF }}
        token: ${{ secrets.WORKFLOW_TOKEN }}
            
    - name: Setup .NET Core SDK
      uses: actions/setup-dotnet@v2.1.0
      with:
        dotnet-version: '6.0.x'

    - name: Install dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --configuration Release --no-restore
    
    - name: Pack Artifact
      run: cd Tutorial.Api/bin/Release/net6.0 && tar -zcvf ${{ env.ARTIFACT_NAME }}.tar.gz *

    - name: Upload Artifact
      uses: actions/upload-artifact@v3
      with:
        name: ${{ env.ARTIFACT_NAME }}
        path: '${{ github.workspace }}/Tutorial.Api/bin/Release/net6.0/${{ env.ARTIFACT_NAME }}.tar.gz'
```

<div align="center"><img src="../img/image-20221206-074258.png" width="90%"></div>

Commit and push code to GitHub repository on main branch.

Go back to repository `<username>-`pipeline` on tab Actions you can see workflow named `DEV - Tutorial BE Deploy` visible

<div align="center"><img src="../img/image-20221206-074318.png" width="90%"></div>

before running this workflow, you must config variable secret at <i>Settings > Environments > dev</i>

if you don't found environment named `dev` you can click create `New environment` button and enter `dev`

<div align="center"><img src="../img/image-20221206-074335.png" width="90%"></div>

click on environment `dev` name to set Environment secrets

Enter

- Name: `WORKFLOW_TOKEN`
- Value: `ghp_xxxxxxxxxxxxxxxxx`

<div align="center"><img src="../img/image-20221206-074629.png" width="90%"></div>

and go back to Actions tab and run workflows `DEV - Tutorial BE Deploy`.

<div align="center"><img src="../img/image-20221206-074717.png" width="90%"></div>


<b>Summary Code</b>

```yaml
name: DEV - Tutorial BE Deploy

on:
  repository_dispatch:
    types: [<username>-tutorial-be-cd-dev]
  workflow_dispatch:
    inputs:
      ref:
        description: "Repository branch or tag"
        required: true
        default: "develop"

env:
  REPOSITORY: "<username>/<username>-tutorial-backend"
  GITREF: ${{ github.event_name == 'repository_dispatch' && 'develop' || github.event.inputs.ref }}
  ARTIFACT_NAME: "artifact-tutorial-be-0.0.1-snapshot"
  APP_RESOURCE_NAME: app-<username>-az-<region>-dev-001

jobs:
  build-app-service:
    name: Build
    runs-on: ubuntu-latest
    environment:
      name: dev
    steps:
    - name: "Checkout project"
      uses: actions/checkout@v2
      with:
        repository: ${{ env.REPOSITORY }}
        ref: ${{ env.GITREF }}
        token: ${{ secrets.WORKFLOW_TOKEN }}
            
    - name: Setup .NET Core SDK
      uses: actions/setup-dotnet@v2.1.0
      with:
        dotnet-version: '6.0.x'

    - name: Install dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --configuration Release --no-restore
    
    - name: Pack Artifact
      run: cd Tutorial.Api/bin/Release/net6.0 && tar -zcvf ${{ env.ARTIFACT_NAME }}.tar.gz *

    - name: Upload Artifact
      uses: actions/upload-artifact@v3
      with:
        name: ${{ env.ARTIFACT_NAME }}
        path: '${{ github.workspace }}/Tutorial.Api/bin/Release/net6.0/${{ env.ARTIFACT_NAME }}.tar.gz'

```

<div align="center"><img src="../img/image-20221206-074810.png" width="90%"></div>

### Deploy

Deploy to Azure Webapp on Environment Dev

```yaml
  deploy-app-service:
    name: Deploy
    runs-on: [ self-hosted, <username>-sbx, vm-<username>SelfHost-az-<region>-sbx-001 ]
    needs: [build-app-service]
    environment:
      name: dev
      url: https://app-<username>-az-<region>-dev-001.azurewebsites.net/swagger/index.html
 
```

#### Step

```yaml
    steps:
    - name: Download Artifact
      uses: actions/download-artifact@v3
      with:
        name: ${{ env.ARTIFACT_NAME }}
        path: ${{ github.workspace }}/${{ env.ARTIFACT_NAME }}
    
    - name: Extract Artifact
      run : |
        tar -zxf ${{ github.workspace }}/${{ env.ARTIFACT_NAME }}/${{ env.ARTIFACT_NAME }}.tar.gz -C ${{ github.workspace }}/${{ env.ARTIFACT_NAME }}
        rm ${{ github.workspace }}/${{ env.ARTIFACT_NAME }}/${{ env.ARTIFACT_NAME }}.tar.gz
    
    - name: Login via Azure CLI
      run: |
        az login --identity --username ${{ secrets.AZURE_SELFHOST_USER_MANAGE_IDENTITY_CLIENT_ID }}
        az account set --subscription ${{ secrets.AZURE_SUBSCRIPTION_ID }}
    
    - name: Deploy to Azure WebApp
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.APP_RESOURCE_NAME }}
        package: ${{ github.workspace }}/${{ env.ARTIFACT_NAME }}
    
    - name: Clear Cache
      run: |
        rm -rf ${{ github.workspace }}/${{ env.ARTIFACT_NAME }}

```

<div align="center"><img src="../img/image-20221206-074905.png" width="90%"></div>

Commit and push code to GitHub repository on main branch.

Go back to repository `<username>-pipeline` on tab Actions you can see workflow named `DEV - Tutorial BE Deploy` visible

<div align="center"><img src="../img/image-20221206-074929.png" width="90%"></div>

before running this workflow, you must config variables secret at Settings > Environments > dev

Enter

- AZURE_SELFHOST_USER_MANAGE_IDENTITY_CLIENT_ID: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
- AZURE_SUBSCRIPTION_ID: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

<div align="center"><img src="../img/image-20221206-075110.png" width="90%"></div>

<i>AZURE_SELFHOST_USER_MANAGE_IDENTITY_CLIENT_ID</i> you can copy ID from resource `id-<username>Selfhost-az-<region>-sbx-001` on Overview page.

<div align="center"><img src="../img/image-20221206-075148.png" width="90%"></div>

And check permission on Azure role assignment before run workflow

<i>if you not found role, please add role assignment in your resource group</i>

<div align="center"><img src="../img/image-20221206-075320.png" width="90%"></div>

Run deploy workflow dev

<div align="center"><img src="../img/image-20221206-075333.png" width="90%"></div>

<div align="center"><img src="../img/image-20221206-075339.png" width="90%"></div>


<b>Summary Code</b>

```yaml
name: DEV - Tutorial BE Deploy

on:
  repository_dispatch:
    types: [<username>-tutorial-be-cd-dev]
  workflow_dispatch:
    inputs:
      ref:
        description: "Repository branch or tag"
        required: true
        default: "develop"

env:
  REPOSITORY: "<username>/<username>-tutorial-backend"
  GITREF: ${{ github.event_name == 'repository_dispatch' && 'develop' || github.event.inputs.ref }}
  ARTIFACT_NAME: "artifact-tutorial-be-0.0.1-snapshot"
  APP_RESOURCE_NAME: app-<username>-az-<region>-dev-001

jobs:
  build-app-service:
    name: Build
    runs-on: ubuntu-latest
    environment:
      name: dev
    steps:
    - name: "Checkout project"
      uses: actions/checkout@v2
      with:
        repository: ${{ env.REPOSITORY }}
        ref: ${{ env.GITREF }}
        token: ${{ secrets.WORKFLOW_TOKEN }}
            
    - name: Setup .NET Core SDK
      uses: actions/setup-dotnet@v2.1.0
      with:
        dotnet-version: '6.0.x'

    - name: Install dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --configuration Release --no-restore
    
    - name: Pack Artifact
      run: cd Tutorial.Api/bin/Release/net6.0 && tar -zcvf ${{ env.ARTIFACT_NAME }}.tar.gz *

    - name: Upload Artifact
      uses: actions/upload-artifact@v3
      with:
        name: ${{ env.ARTIFACT_NAME }}
        path: '${{ github.workspace }}/Tutorial.Api/bin/Release/net6.0/${{ env.ARTIFACT_NAME }}.tar.gz'

  deploy-app-service:
    name: Deploy
    runs-on: [ self-hosted, <username>-sbx, vm-<username>SelfHost-az-<region>-sbx-001 ]
    needs: [build-app-service]
    environment:
      name: dev
      url: https://app-<username>-az-asse-dev-001.azurewebsites.net/swagger/index.html
    
    steps:
    - name: Download Artifact
      uses: actions/download-artifact@v3
      with:
        name: ${{ env.ARTIFACT_NAME }}
        path: ${{ github.workspace }}/${{ env.ARTIFACT_NAME }}
    
    - name: Extract Artifact
      run : |
        tar -zxf ${{ github.workspace }}/${{ env.ARTIFACT_NAME }}/${{ env.ARTIFACT_NAME }}.tar.gz -C ${{ github.workspace }}/${{ env.ARTIFACT_NAME }}
        rm ${{ github.workspace }}/${{ env.ARTIFACT_NAME }}/${{ env.ARTIFACT_NAME }}.tar.gz
    
    - name: Login via Azure CLI
      run: |
        az login --identity --username ${{ secrets.AZURE_SELFHOST_USER_MANAGE_IDENTITY_CLIENT_ID }}
        az account set --subscription ${{ secrets.AZURE_SUBSCRIPTION_ID }}

    - name: Deploy to Azure WebApp
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.APP_RESOURCE_NAME }}
        package: ${{ github.workspace }}/${{ env.ARTIFACT_NAME }}
    
    - name: Clear Cache
      run: |
        rm -rf ${{ github.workspace }}/${{ env.ARTIFACT_NAME }}

```

<b>Logging</b>

<div align="center"><img src="../img/image-20221206-075408.png" width="90%"></div>


## Create DEV - Configuration Set workflows

On Create DEV - Tutorial BE Deploy workflows is working on workflows CI Dev dispatch and automate deploy to Azure Webapp.

<div align="center"><img src="../img/image-20221206-075440.png" width="30%"></div>


Create the new file named dev-tutorial-be-configuration-set.yml inside `.github/workflows` this on the Workflows which contains configure to App service (Webapp)

<div align="center"><img src="../img/image-20221206-075613.png" width="90%"></div>

### Name

The first name starts with declare name of workflows

```yaml
name: DEV - Tutorial BE Configuration
```

### On (Events that trigger workflows)

Enter events when do you want to execute or trigger the workflows

you can see more of event type at [Events that trigger workflows - GitHub Docs](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

```yaml
on:
  workflow_dispatch:
```

Jobs
Groups together all the jobs that run in the `DEV - Tutorial BE Deploy` workflow.

```yaml
jobs:
...
```

<b>Set Configuration</b>

The build artifact to contains in the job

```yaml
  set-configuration:
    name: set appservice configuration
    runs-on: [ self-hosted, tutorial-be-dev, vm-<username>SelfHost-az-<region>-sbx-001 ]
    environment: dev

```

<b>Steps</b>

```yaml
    steps:
    - name: Login via Azure CLI
      run: |
        az login --identity --username ${{ secrets.AZURE_SELFHOST_USER_MANAGE_IDENTITY_CLIENT_ID }}
        az account set --subscription ${{ secrets.AZURE_SUBSCRIPTION_ID }}

    - name: Set Web App Settings
      uses: Azure/appservice-settings@v1
      with:
        app-name: app-<username>-az-<region>-dev-001
        app-settings-json: |
          [
            {
              "name": "TutorialDatabase__ConnectionString",
              "value": "${{ secrets.TUTORIAL_DB_CONNECTION_STRING }}",
              "slotSetting": false
            },
            {
              "name": "TutorialDatabase__DatabaseName",
              "value": "dev-tutorial",
              "slotSetting": false
            },
            {
              "name": "TutorialDatabase__TutorialCollectionName",
              "value": "tutorials",
              "slotSetting": false
            }
          ]

```

<div align="center"><img src="../img/image-20221206-075707.png" width="90%"></div>

Commit and push code to GitHub repository on `main` branch.

Go back to repository `<username>-pipeline` on tab Actions you can see workflow named `DEV - Tutorial BE Configuration` visible

<div align="center"><img src="../img/image-20221206-075727.png" width="90%"></div>

before running this workflow, you must config variable secret at Settings > Environments > dev

<b>Enter</b>
- Name : `TUTORIAL_DB_CONNECTION_STRING`
- Value : `mongodb://...`

<div align="center"><img src="../img/image-20221206-075837.png" width="90%"></div>

And go back to Actions tab and run workflows `DEV - Tutorial BE Configuration`.

<div align="center"><img src="../img/image-20221206-075921.png" width="90%"></div>

<div align="center"><img src="../img/image-20221206-075933.png" width="90%"></div>

Result Output to set on Configuration Webapp.

<div align="center"><img src="../img/image-20221206-075947.png" width="90%"></div>


Try to access via URL `https://app-username-az-<region>-dev-001.azurewebsites.net>/swagger/index.html`

<div align="center"><img src="../img/image-20221206-080050.png" width="90%"></div>

<b>Summary Code<b>

```yaml
name: DEV - Tutorial BE Configuration

on:
  workflow_dispatch:

jobs:
  set-configuration:
    name: set appservice configuration
    runs-on: [ self-hosted, <username>-sbx, vm-<username>SelfHost-az-<region>-sbx-001 ]
    environment: dev
    steps:
    - name: Login via Azure CLI
      run: |
        az login --identity --username ${{ secrets.AZURE_SELFHOST_USER_MANAGE_IDENTITY_CLIENT_ID }}
        az account set --subscription ${{ secrets.AZURE_SUBSCRIPTION_ID }}

    - name: Set Web App Settings
      uses: Azure/appservice-settings@v1
      with:
        app-name: app-<username>-az-<region>-dev-001
        app-settings-json: |
          [
            {
              "name": "TutorialDatabase__ConnectionString",
              "value": "${{ secrets.TUTORIAL_DB_CONNECTION_STRING }}",
              "slotSetting": false
            },
            {
              "name": "TutorialDatabase__DatabaseName",
              "value": "dev-tutorial",
              "slotSetting": false
            },
            {
              "name": "TutorialDatabase__TutorialCollectionName",
              "value": "tutorials",
              "slotSetting": false
            }
          ]
          

```

<b>Logging</b>

<div align="center"><img src="../img/image-20221206-080108.png" width="90%"></div>


Done :D
