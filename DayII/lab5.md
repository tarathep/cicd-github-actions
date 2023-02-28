# LAB 5 : Continuous Integration with GitHub Actions

Learn how continuous integrate application working on GitHub Actions and implementation basic concepts.

After completing this lab, you'll be able to:
- Explain and Implement CI workflows with GitHub Actions in fundamental.
- Explain Branching strategy working with each environment.
- Automation workflows and how to use Actions.
- Investigate and solvable working on pipeline.

## Prerequisites

- <b>Workspace that required Software and Tools</b> 
    - Git and GitHub Account
    - Text Editor (Required Visual Studio Code, or Visual Studio) [Visual Studio Code - Code Editing. Redefined](https://code.visualstudio.com/)
   
- Requred lab4

## Workflows Diagram

<div align="center"><img src="../img/image-20221123-033023.png" width="90%"></div>

## Initialize GitHub workflow

Checkout the source code from GitHub from here `<username>/<username>-tutorial-backend`.

Open the terminal following command below

```bash
git clone https://github.com/<username>/<username>-tutorial-backend.git
```

Open the project with text editor (Visual Studio Code) and create new folder named `github/workflows` in root directory of project (git).

```bash
mkdir .github/workflows
```

<div align="center"><img src="../img/image-20221123-034740.png" width="90%"></div>

GitHub workflow is working on inside `.github/workflows` that contains GitHub workflows files that extension named `.yaml`

## Create Develop workflows

On develop workflows branch is working on branch `develop` and automate by developer when push code to GitHub repository on branch `develop`.

<div align="center"><img src="../img/image-20221123-035134.png" width="90%"></div>

Create the new file named `cicd-appservice-dev.yaml` inside `.github/workflows` this on the Workflows Dev which contains 3 workflows: <B>Unit Tests, SAST and DEV Dispatch</b>.

<div align="center"><img src="../img/image-20221123-040051.png" width="90%"></div>


### Name

The first name starts with declare name of workflows

```yaml
name: DEV - Dispatch
``` 

### On (Events that trigger workflows)

Enter events when do you want to execute or trigger the workflows

you can see more of event type at [Events that trigger workflows - GitHub Docs](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

```yaml
on: 
  push:
    branches:    
      - develop
  workflow_dispatch:
```

 

### Jobs

Groups together all the jobs that run in the DEV - Dispatch workflow.

```yaml
jobs:
...
```

---

### Unit Tests

The unit tests contain in the job

```yaml
  unitest:
    name: UnitTest (XUnit)
    runs-on: ubuntu-latest
    environment:
      name: dev

```

#### Steps

```yaml
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Setup .NET Core SDK
      uses: actions/setup-dotnet@v2.1.0
      with:
        dotnet-version: '6.0.x'

    - name: Install dependencies
      run: dotnet restore

    - name: Unit Test (XUnit)
      run: |
        dotnet test --collect:"XPlat Code Coverage"
        dotnet tool install -g dotnet-reportgenerator-globaltool
        reportgenerator -reports:"./XUnit.Tests/TestResults/*/coverage.cobertura.xml" -targetdir:"./coveragereport" -reporttypes:Html
    
    - name: Upload Reports
      uses: actions/upload-artifact@v2
      with:
        name: Unit Test Results
        path: '${{ github.workspace }}/coveragereport/*'

```

<div align="center"><img src="../img/image-20221123-044202.png" width="90%"></div>

Commit and push code to GitHub repository on main branch.

<div align="center"><img src="../img/image-20221123-044545.png" width="90%"></div>


And then go to the Actions tab in the actions page you can see `DEV -Dispatch` workflow

<div align="center"><img src="../img/image-20221123-044656.png" width="90%"></div>

you can test this workflow to create new branch named `develop` from `main` branch.

On the local workspace create new branch with command

```bash
git branch develop
```

```bash
git checkout develop
```

<div align="center"><img src="../img/image-20221123-045031.png" width="90%"></div>

and push this new branch named `develop` to GitHub repository, for the first new branch use this command in below.

```bash
git push --set-upstream origin develop
```

Go back to repository on GitHub page, you can see the new branch named `develop`.

Go to the settings repository to set `default branch to develop` branch

<div align="center"><img src="../img/image-20221123-045440.png" width="90%"></div>

In panel Code and automation > choose Branches > Click on icon arrow right left button.

Popup dialog show and then change from main to develop at dropdown list button and `update` it.

<div align="center"><img src="../img/image-20221123-045740.png" width="90%"></div>

Go back to Actions tab, you can see the pipeline is running.

<div align="center"><img src="../img/image-20221123-050413.png" width="90%"></div>

<div align="center"><img src="../img/image-20221123-050551.png" width="90%"></div>

<div align="center"><img src="../img/image-20221123-050612.png" width="90%"></div>

On Artifacts, you can download Unit Test Results file to review reports of application

<div align="center"><img src="../img/image-20221123-050917.png" width="90%"></div>

we are working on branch develop.

<b>Summary Code</b>

```yaml
name: DEV - Dispatch

on: 
  push:
    branches:    
      - develop
  workflow_dispatch:

jobs:
  unitest:
    name: UnitTest (XUnit)
    runs-on: ubuntu-latest
    environment:
      name: dev
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Setup .NET Core SDK
      uses: actions/setup-dotnet@v2.1.0
      with:
        dotnet-version: '6.0.x'

    - name: Install dependencies
      run: dotnet restore

    - name: Unit Test (XUnit)
      run: |
        dotnet test --collect:"XPlat Code Coverage"
        dotnet tool install -g dotnet-reportgenerator-globaltool
        reportgenerator -reports:"./XUnit.Tests/TestResults/*/coverage.cobertura.xml" -targetdir:"./coveragereport" -reporttypes:Html
    
    - name: Upload Reports
      uses: actions/upload-artifact@v2
      with:
        name: Unit Test Results
        path: '${{ github.workspace }}/coveragereport/*'

```

<b>logging</b>

<div align="center"><img src="../img/image-20221123-063125.png" width="90%"></div>

---

### SAST (Static Application Security Testing)

Go back to workspace on local and add new job

```yaml
  analyze:
    name: SAST CodeQL
    runs-on: ubuntu-latest
    environment:
      name: dev
      url: https://github.com/<owner>/<username>-tutorial-backend/security/code-scanning
    permissions:
      actions: read
      contents: read
      security-events: write

    strategy:
      fail-fast: false
      matrix:
        language: [ 'csharp' ]
```

#### Steps

```yaml
    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Initialize CodeQL
      uses: github/codeql-action/init@v2
      with:
        languages: ${{ matrix.language }}
       
    - name: Autobuild
      uses: github/codeql-action/autobuild@v2

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2

    # THIS IS INTERNAL ACITON ACCESS ONLY INTERNAL OGANIZAITON
    # - name: Quality Gate Check
    #   uses: <owner>/quality-gate-action@main
    #   with:
    #     repository: <owner>/xxx-xxxxxx
    #     severity: high
    #   env:
    #     GITHUB_TOKEN: ${{ secrets.WORKFLOW_TOKEN }}
```

Commit and push code to GitHub repository on develop branch.

<div align="center"><img src="../img/image-20221123-062738.png" width="90%"></div>

Go back to Actions tab, you can see the pipeline is running.

<div align="center"><img src="../img/image-20221123-062823.png" width="90%"></div>

Review output in Security tab on top.

On Security overview, click to enable on the first time.
- Enable vulnerability reporting
- Enable Dependabot alerts

<div align="center"><img src="../img/image-20221123-063731.png" width="90%"></div>

<div align="center"><img src="../img/image-20221123-063956.png" width="90%"></div>

Go back to Security tab and go to output in left corner in Vulnerability alerts.

### Dependabot

on `Dependabot` is active when upload source code in GitHub repository

<div align="center"><img src="../img/image-20221123-064304.png" width="90%"></div>


### Code scanning with CodeQL

Code scanning is active when running the SAST workflow.

<div align="center"><img src="../img/image-20221123-064647.png" width="90%"></div>


<b>Summary Code</b>

```yaml
name: DEV - Dispatch

on: 
  push:
    branches:    
      - develop
  workflow_dispatch:

jobs:
  unitest:
    name: UnitTest (XUnit)
    runs-on: ubuntu-latest
    environment:
      name: dev
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Setup .NET Core SDK
      uses: actions/setup-dotnet@v2.1.0
      with:
        dotnet-version: '6.0.x'

    - name: Install dependencies
      run: dotnet restore

    - name: Unit Test (XUnit)
      run: |
        dotnet test --collect:"XPlat Code Coverage"
        dotnet tool install -g dotnet-reportgenerator-globaltool
        reportgenerator -reports:"./XUnit.Tests/TestResults/*/coverage.cobertura.xml" -targetdir:"./coveragereport" -reporttypes:Html
    
    - name: Upload Reports
      uses: actions/upload-artifact@v2
      with:
        name: Unit Test Results
        path: '${{ github.workspace }}/coveragereport/*'

  analyze:
    name: SAST CodeQL
    runs-on: ubuntu-latest
    environment:
      name: dev
      url: https://github.com/<username>/<username>-tutorial-backend/security/code-scanning
    permissions:
      actions: read
      contents: read
      security-events: write

    strategy:
      fail-fast: false
      matrix:
        language: [ 'csharp' ]

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Initialize CodeQL
      uses: github/codeql-action/init@v2
      with:
        languages: ${{ matrix.language }}
       
    - name: Autobuild
      uses: github/codeql-action/autobuild@v2


    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2

    # THIS IS INTERNAL ACITON ACCESS ONLY INTERNAL OGANIZAITON
    # - name: Quality Gate Check
    #   uses: <owner>/quality-gate-action@main
    #   with:
    #     repository: <owner>/<this-repos-name>
    #     severity: high
    #   env:
    #     GITHUB_TOKEN: ${{ secrets.WORKFLOW_TOKEN }}

```

<b>Logging</b>

<div align="center"><img src="../img/image-20221123-072012.png" width="90%"></div>

---

### Repository Dispatch

Go back to workspace on local and add new job

```yaml
  repo-dispatch:
    name: Repository Dispatch
    runs-on: ubuntu-latest
    needs: [unitest,analyze]
    environment:
      name: dev
      url: https://github.com/<username>/<username>-pipeline/actions/workflows/dev-tutorial-backend-deploy.yml
```

### Steps

```yaml
    steps:
    - name: Repository Dispatch
      uses: peter-evans/repository-dispatch@v1
      with:
        token: ${{ secrets.WORKFLOW_TOKEN }}
        repository: <owner>/<username>-pipeline
        event-type: <username>-tutorial-be-cd-dev
        client-payload: '{"ref": "${{ github.ref }}", "sha": "${{ github.sha }}"}'
```

in the variable secrets, you must set env <i>Settings page > Environments > dev > Add secret and enter personal token</i> that have permission to allow access repository level.

<div align="center"><img src="../img/image-20221123-070535.png" width="90%"></div>

Alternative, you can use reference GITHUB_TOKEN to access and allow adding permission to repository CD.

<div align="center"><img src="../img/image-20221123-071607.png" width="90%"></div>

Commit and push code to GitHub repository on branch develop

Review Output on GitHub Actions page

<div align="center"><img src="../img/image-20221123-072124.png" width="90%"></div>


<b>Summary Code (Dev)</b>

```yaml
name: DEV - Dispatch

on: 
  push:
    branches:    
      - develop
  workflow_dispatch:

jobs:
  unitest:
    name: UnitTest (XUnit)
    runs-on: ubuntu-latest
    environment:
      name: dev
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Setup .NET Core SDK
      uses: actions/setup-dotnet@v2.1.0
      with:
        dotnet-version: '6.0.x'

    - name: Install dependencies
      run: dotnet restore

    - name: Unit Test (XUnit)
      run: |
        dotnet test --collect:"XPlat Code Coverage"
        dotnet tool install -g dotnet-reportgenerator-globaltool
        reportgenerator -reports:"./XUnit.Tests/TestResults/*/coverage.cobertura.xml" -targetdir:"./coveragereport" -reporttypes:Html
    
    - name: Upload Reports
      uses: actions/upload-artifact@v2
      with:
        name: Unit Test Results
        path: '${{ github.workspace }}/coveragereport/*'

  analyze:
    name: SAST CodeQL
    runs-on: ubuntu-latest
    environment:
      name: dev
      url: https://github.com/<username>/<username>-tutorial-backend/security/code-scanning
    permissions:
      actions: read
      contents: read
      security-events: write

    strategy:
      fail-fast: false
      matrix:
        language: [ 'csharp' ]

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Initialize CodeQL
      uses: github/codeql-action/init@v2
      with:
        languages: ${{ matrix.language }}
       
    - name: Autobuild
      uses: github/codeql-action/autobuild@v2

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2

    # THIS IS INTERNAL ACITON ACCESS ONLY INTERNAL OGANIZAITON
    # - name: Quality Gate Check
    #   uses: <owner>/quality-gate-action@main
    #   with:
    #     repository: <owner>/xxx-xxxxxx
    #     severity: high
    #   env:
    #     GITHUB_TOKEN: ${{ secrets.WORKFLOW_TOKEN }}

  repo-dispatch:
    name: Repository Dispatch
    runs-on: ubuntu-latest
    needs: [unitest,analyze]
    environment:
      name: dev
      url: https://github.com/<username>/<username>-tutorial-backend/actions/workflows/dev-tutorial-backend-deploy.yml

    steps:
    - name: Repository Dispatch
      uses: peter-evans/repository-dispatch@v1
      with:
        token: ${{ secrets.WORKFLOW_TOKEN }}
        repository: <username>/<username>-pipeline
        event-type: <username>-tutorial-backend-cd-dev
        client-payload: '{"ref": "${{ github.ref }}", "sha": "${{ github.sha }}"}'
```

<b>Logging</b>

<div align="center"><img src="../img/image-20221123-072225.png" width="90%"></div>

done merge code from branch develop to main .

---

## Create SIT workflows

On the SIT Build workflows is working on main branch and automate trigger by developer when tagging to GitHub repository.

<div align="center"><img src="../img/image-20221123-072802.png" width="90%"></div>

Create the new file named `cicd-appservice-sit.yaml` inside `.github/workflows` this on the Workflows SIT which contains 1 workflow to build artifacts to prepare to deploy.

<div align="center"><img src="../img/image-20221123-073422.png" width="90%"></div>


### Name

The first name starts with declare name of workflows

```yaml
name: SIT - Build
```

### On (Events that trigger workflows)

Enter events when do you want to execute or trigger the workflows

you can see more of event type at Events that [trigger workflows - GitHub Docs](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

```yaml
on:
  push:
    tags:
      - '*'
```

### Env

the environment global to declaration and variables

```yaml
env:
  ARTIFACT_NAME: "artifact-tutorial-backend"
```

### Jobs

Groups together all the jobs that run in the `DEV - Dispatch` workflow.

```yaml
jobs:
...
```

---

### Build

The build artifact to contains in the job

```yaml
  build:
    runs-on: ubuntu-latest
    environment:
      name: sit
    permissions:
      contents: write
```

#### Steps

```yaml

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set env
      if: startsWith(github.ref, 'refs/tags/')
      run: |
        echo "RELEASE_VERSION=${GITHUB_REF#refs/*/}" >> $GITHUB_ENV
    
    - name: Setup .NET Core SDK
      uses: actions/setup-dotnet@v2.1.0
      with:
        dotnet-version: '6.0.x'

    - name: Install dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --configuration Release --no-restore
    
    - name: Pack Artifact
      run: cd Tutorial.Api/bin/Release/net6.0 && tar -zcvf ${{ env.ARTIFACT_NAME }}-${{ env.RELEASE_VERSION }}.tar.gz *
      
    - name: Upload Artifact Release
      uses: softprops/action-gh-release@v1
      with:
        files: ${{ github.workspace }}/Tutorial.Api/bin/Release/net6.0/${{ env.ARTIFACT_NAME }}-${{ env.RELEASE_VERSION }}.tar.gz
```

<div align="center"><img src="../img/image-20221123-075317.png" width="90%"></div>

Commit and push code to GitHub repository on develop branch.

<div align="center"><img src="../img/image-20221123-075457.png" width="90%"></div>

Before your merge `develop` branch to `main` branch, you may set `review and approve`.

Go to in the settings tab on top > in below Code and automation select Branches > click add branch protection rule

<div align="center"><img src="../img/image-20221123-080455.png" width="90%"></div>

In Branch name pattern, enter name: main and checked in below

<div align="center"><img src="../img/image-20221123-080819.png" width="90%"></div>

<div align="center"><img src="../img/image-20221123-080905.png" width="90%"></div>

Go to `Pull requests` tab on top to merge branch from `develop` to `main` branch by click New pull request button

<div align="center"><img src="../img/image-20221123-075652.png" width="90%"></div>

<div align="center"><img src="../img/image-20221123-080144.png" width="90%"></div>

`Merge pull request`, you can see Error for merging is block because in this repository not working with others (assign yourself)

<div align="center"><img src="../img/image-20221123-081510.png" width="90%"></div>

<div align="center"><img src="../img/image-20221123-081852.png" width="90%"></div>

Go back to `Code` tab and create `release to tags`

<div align="center"><img src="../img/image-20221123-082332.png" width="90%"></div>

click Create a new release

<div align="center"><img src="../img/image-20221123-082412.png" width="90%"></div>

Choose a tag enter version following Tagging suggestions (Semantic versioning) and +Create new tag and Target at main branch

<div align="center"><img src="../img/image-20221123-082551.png" width="90%"></div>

click `Publish release` and then go to the Actions tab in the actions page you can see `DEV - Build` workflow

<div align="center"><img src="../img/image-20221123-083542.png" width="90%"></div>

<div align="center"><img src="../img/image-20221123-083703.png" width="90%"></div>

### Output the artifact

go back to the Tags page

<div align="center"><img src="../img/image-20221123-083828.png" width="90%"></div>

<div align="center"><img src="../img/image-20221123-083910.png" width="90%"></div>


You can click to download to review.

<div align="center"><img src="../img/image-20221123-084117.png" width="90%"></div>


<b>Summary Code (SIT)</b>

```yaml
name: SIT - Build

on:
  push:
    tags:
      - '*'

env:
  ARTIFACT_NAME: "artifact-tutorial-backend"

jobs:
  build:
    runs-on: ubuntu-latest
    environment:
      name: sit
    permissions:
      contents: write
      
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set env
      if: startsWith(github.ref, 'refs/tags/')
      run: |
        echo "RELEASE_VERSION=${GITHUB_REF#refs/*/}" >> $GITHUB_ENV
    
    - name: Setup .NET Core SDK
      uses: actions/setup-dotnet@v2.1.0
      with:
        dotnet-version: '6.0.x'

    - name: Install dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --configuration Release --no-restore
    
    - name: Pack Artifact
      run: cd Tutorial.Api/bin/Release/net6.0 && tar -zcvf ${{ env.ARTIFACT_NAME }}-${{ env.RELEASE_VERSION }}.tar.gz *
      
    - name: Upload Artifact Release
      uses: softprops/action-gh-release@v1
      with:
        files: ${{ github.workspace }}/Tutorial.Api/bin/Release/net6.0/${{ env.ARTIFACT_NAME }}-${{ env.RELEASE_VERSION }}.tar.gz
```

<b>logging</b>

<div align="center"><img src="../img/image-20221123-084315.png" width="90%"></div>

Done :D














