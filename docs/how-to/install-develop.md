# Install a development MonoGame build

Do you want to build your game with MonoGame's latest and greatest develop branch? You don't need to wait for official MonoGame releases.

Each time a new commit makes it into the MonoGame development branch, new builds are automatically released to the [MonoGame TeamCity](http://teamcity.monogame.net/viewType.html?buildTypeId=MonoGame_DevelopWin&branch_MonoGame=%3Cdefault%3E&tab=buildTypeStatusDiv&guest=1) site. To install them into a project, you can execute the following commands.

1. Add the MonoGame teamcity NuGet feed:
   ```
   dotnet nuget add source -n MonoGame http://teamcity.monogame.net/guestAuth/app/nuget/feed/_Root/default/v3/index.json
   ```
2. From the folder with your game's .csproj:
   ```
   dotnet add package MonoGame.Framework.DesktopGL --prerelease
   dotnet add package MonoGame.Content.Builder.Task --prerelease
   ```

You can repeat step 2 whenever you want to want to update.

Note that sometimes teamcity is down. When that happens the commands can fail.
