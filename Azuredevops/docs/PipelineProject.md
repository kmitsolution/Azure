![image](https://user-images.githubusercontent.com/84008107/127195244-d4a3d874-9c84-46eb-bc04-e4d0e554daa1.png)

#### Steps to create Repo and push to Repo

dotnet new sln -o HelloWorldApp

cd HelloWorldApp

dotnet new mvc -n HelloWorldApp.Web

dotnet sln HelloWorldApp.sln add HelloWorldApp.Web\HelloWorldApp.Web.csproj

dotnet restore

dotnet build --no-restore --configuration release

dotnet publish --no-build --configuration release

cd HelloWorldApp.Web

dotnet C:\Temp\azuredevops\HelloWorldApp\HelloWorldApp.Web\bin\Release\net5.0\HelloWorldApp.Web.dll

Goto browser and localhost:5000\

code .


#### Create a project in Organization (Azure Devops) and make a private project

1. Lets create a Repo <br />
2. Push an existing repository from command line and copy Https URL <br />
3. Goto command line and cd  C:\Temp\azuredevops\HelloWorldApp <br />
4. Initialize git with <br />
git init <br />
5. add a .gitignore file for visual studio (https://github.com/github/gitignore/blob/master/VisualStudio.gitignore) <br />
6. git add . <br />
7. git commit -m "version1 of code is added" <br />
8. git push -u origin --all <br />
