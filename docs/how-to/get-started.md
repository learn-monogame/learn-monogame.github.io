# Get started

This guide will give you the steps to install MonoGame, get a basic project setup, and build an output that you can publish.

## Install

1. Get the [.NET Core 3.1 SDK](https://dotnet.microsoft.com/download).
   * Test that dotnet is installed correctly:
    ```
    dotnet --version
    ```
2. Get the MonoGame templates:
    ```
    dotnet new --install MonoGame.Templates.CSharp
    ```
3. Get the MonoGame Content Pipeline:
    ```
    dotnet tool install --global dotnet-mgcb-editor
    mgcb-editor --register
    ```

## Create a new game

Replace `MyGame` with your game's name in the following two command:

```
dotnet new mgdesktopgl -o MyGame
cd MyGame
```

## Run

```
dotnet run
```

## Publish

You can publish on Windows, Mac, and Linux using:

```
dotnet publish -c Release -r win-x64 -o artifacts/windows
dotnet publish -c Release -r osx-x64 -o artifacts/osx
dotnet publish -c Release -r linux-x64 -o artifacts/linux
```

You'll find the output in:

```
artifacts/windows
artifacts/osx
artifacts/linux
```

You can zip those folders to share your game.

## Bug

In MonoGame 3.8.0.1641 there's a bug that prevents you from compiling your game if you have a space anywhere in the build path. There is an easy manual fix thankfully. Apply the fix found [here](https://github.com/MonoGame/MonoGame/commit/886b639294a357e11e8afd601c4fe1dac9ad6ef2#diff-c8381b2ce7b41ae4345cf3064ab209be946637813f51ae6e5656dae7a7a48ab9). Find the [global package location](https://docs.microsoft.com/en-us/nuget/consume-packages/managing-the-global-packages-and-cache-folders) for your system. Edit the .targets file at: `~/.nuget/packages/monogame.content.builder.task/3.8.0.1641/build/MonoGame.Content.Builder.Task.targets`. Surround `$(MGCBPath)` with `&quot;` on line 140:

```xml {lineStart:140}
Command="$(DotnetCommand) &quot;$(MGCBPath)&quot; $(MonoGameMGCBAdditionalArguments) /@:&quot;%(ContentReference.FullPath)&quot; /platform:$(MonoGamePlatform) /outputDir:&quot;%(ContentReference.ContentOutputDir)&quot; /intermediateDir:&quot;%(ContentReference.ContentIntermediateOutputDir)&quot; /workingDir:&quot;%(ContentReference.FullDir)&quot;"
```

## Read more

You can read more getting started info from the official [MonoGame docs](https://docs.monogame.net/articles/getting_started/0_getting_started.html).

---

Now that you have a project, the next step is to setup a development environment. [Developing with Visual Studio Code](./develop-vscode/README.md).
