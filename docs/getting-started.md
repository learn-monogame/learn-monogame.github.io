# Getting Started

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

Replace MyGame with your game's name in the follow command:

```
dotnet new mgdesktopgl -o MyGame
```

## Run

```
dotnet run
```

## Publish

You can publish on Windows, Mac, and Linux using:

```
dotnet publish -r win-x64 -c Release
dotnet publish -r osx-x64 -c Release
dotnet publish -r linux-x64 -c Release
```

You'll find the output in:

```
bin/Release/netcoreapp3.1/win-x64/publish
bin/Release/netcoreapp3.1/osx-x64/publish
bin/Release/netcoreapp3.1/linux-x64/publish
```

## Read more

You can read more getting started info from the official [MonoGame docs](https://docs.monogame.net/articles/getting_started/0_getting_started.html).
