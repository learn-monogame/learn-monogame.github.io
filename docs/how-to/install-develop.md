# Install a development MonoGame build

Do you want to build your game with MonoGame's latest and greatest develop branch? You don't need to wait for official MonoGame releases.

Each time a new commit makes it into the [MonoGame development branch](https://github.com/MonoGame/MonoGame/commits/develop), new builds are automatically released to the [MonoGame TeamCity](http://teamcity.monogame.net/viewType.html?buildTypeId=MonoGame_DevelopWin&branch_MonoGame=%3Cdefault%3E&tab=buildTypeStatusDiv&guest=1) site. To install them into a project, you can execute the following commands. Note that some of them can take some time to complete as large files are downloaded.

1. Add the MonoGame teamcity NuGet feed:
   ```
   dotnet nuget add source -n MonoGame http://teamcity.monogame.net/guestAuth/app/nuget/feed/_Root/default/v3/index.json
   ```
2. From the folder with your game's .csproj:
   ```
   dotnet add package MonoGame.Framework.DesktopGL --prerelease
   dotnet add package MonoGame.Content.Builder.Task --prerelease
   ```
3. To install the MonoGame Content Builder Editor:
   ```
   dotnet tool update --global dotnet-mgcb-editor --version *-*
   mgcb-editor --register
   ```
4. To install the MonoGame templates:
   ```
   dotnet new --install MonoGame.Templates.CSharp::*-*
   ```
   With that you will be able to use `dotnet new mgdesktopgl -o MyGame`.

You can repeat step 2, 3, and 4 whenever you want to update.

Note that sometimes teamcity is down. When that happens the commands can fail.

You can also install a specific version:

```
dotnet add package MonoGame.Framework.DesktopGL --version 3.8.1.1983-develop
dotnet add package MonoGame.Content.Builder.Task --version 3.8.1.1983-develop
dotnet tool update --global dotnet-mgcb-editor --version 3.8.1.1983-develop
mgcb-editor --register
dotnet new --install MonoGame.Templates.CSharp::3.8.1.1983-develop
```
