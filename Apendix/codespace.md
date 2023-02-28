# Prepare Environment with Codespace on GitHub

go to the https://github.com (logined)

1. [Create a new repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository) named "Workspace".

2. On the top side you can see <b>Codespaces</b>

<div align="center"><img src="../img/a-1.jpeg" width="90%"></div>

3. Click New codespace

<div align="center"><img src="../img/a-2.jpeg" width="90%"></div>

4. Choose 
    - Your Repository named `<username>/workspace`
    - Branch : main
    - Region : Default
    - Machine type : 2-core
and then `Create codespace`

<div align="center"><img src="../img/a-3.jpeg" width="90%"></div>

5. You can see Visual Studio Code.

<div align="center"><img src="../img/a-4.png" width="90%"></div>


## Setting up a C# (.NET) project for GitHub Codespaces

1. Prepare Dotnet and Azure CLI press `Ctrl` + `Shift` + `P`, then start typing <b>"dev container"</b>. Click `Codespaces: Configure Dev Container`.

<div align="center"><img src="../img/a-5.png" width="90%"></div>

2. Click `Create a new configuration..`

<div align="center"><img src="../img/a-6.png" width="90%"></div>

3. Click Show All Definitions.

4. Type c# and click C# (.NET)

<div align="center"><img src="../img/a-7.png" width="90%"></div>

5. Choose the version of .NET you want to use for your project. In this case, select the version marked "6.0"

<div align="center"><img src="../img/a-8.png" width="90%"></div>

6. A list of additional features is displayed. We'll install the .NET CLI, a command-line interface for developing, building, running, and publishing .NET applications. To install this tool, type dotnet, select Dotnet CLI and Azure CLI then click OK.

<div align="center"><img src="../img/a-9.png" width="90%"></div>

<div align="center"><img src="../img/a-10.png" width="90%"></div>

A message is displayed telling you that the dev container configuration file already exists. Click Overwrite.

7. A devcontainer.json file is created and is opened in the editor.

<div align="center"><img src="../img/a-11.jpeg" width="90%"></div>



8. Access the VS Code Command Palette `(Shift+Command+P / Ctrl+Shift+P)`, then start typing <b>"rebuild"</b>. Click `Codespaces: Rebuild Container.

<div align="center"><img src="../img/a-12.png" width="90%"></div>


9. Rebuild

<div align="center"><img src="../img/a-13.jpeg" width="90%"></div>


Done.

Ref. : https://docs.github.com/en/codespaces/setting-up-your-project-for-codespaces/adding-a-dev-container-configuration/setting-up-your-dotnet-project-for-codespaces#step-2-add-a-dev-container-configuration








