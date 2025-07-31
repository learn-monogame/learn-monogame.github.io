# Install a development MonoGame build

Do you want to build your game with MonoGame's latest and greatest develop branch? You don't need to wait for official MonoGame releases.

Each time a new commit makes it into the [MonoGame develop branch](https://github.com/MonoGame/MonoGame/commits/develop), new builds are automatically released to the [MonoGame GitHub NuGet feed](https://github.com/orgs/MonoGame/packages?repo_name=MonoGame). To install them into a project, you can execute the following commands. Note that some of them can take some time to complete as large files are downloaded.

1. Generate a Personal access tokens (classic): <https://github.com/settings/tokens>.
   * Make sure that it has the `read:packages` scope.
2. Add the MonoGame GitHub NuGet feed. Replace `USERNAME` with your GitHub username and `TOKEN` with the generated access token from above:
   ```
   dotnet nuget add source -n MonoGameGitHub https://nuget.pkg.github.com/MonoGame/index.json --username USERNAME --password TOKEN
   ```
3. From the folder with your game's .csproj:
   ```
   dotnet add package MonoGame.Framework.DesktopGL --prerelease
   dotnet add package MonoGame.Content.Builder.Task --prerelease
   ```
4. To install the MonoGame templates:
   ```
   dotnet new install MonoGame.Templates.CSharp::*-*
   ```
   With that you will be able to use `dotnet new mgdesktopgl -o MyGame`.

You can repeat step 3 and 4 whenever you want to update.

You can also install a specific version:

```
dotnet add package MonoGame.Framework.DesktopGL --version 3.8.4.2602-develop
dotnet add package MonoGame.Content.Builder.Task --version 3.8.4.2602-develop
dotnet new install MonoGame.Templates.CSharp::3.8.4.2602-develop
```

To remove the GitHub source from step 2, you can do:

```
dotnet nuget remove source MonoGameGitHub
```

## Read more

* [Working with the NuGet registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry)
* [Managing your personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
* [dotnet nuget add source](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-nuget-list-source)
* [dotnet nuget list source](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-nuget-list-source)
