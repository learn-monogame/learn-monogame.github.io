# Customize the game icon
It takes 2 seconds to set up a custom icon!

This guide is a follow up to [Get started](./get-started.md).

## Instructions

Open your `MyGame.csproj`, search for the following line:

```xml
<EmbeddedResource Include="Icon.bmp" />
```

replace it with:

```xml
<EmbeddedResource Include="Icon.bmp">
  <LogicalName>Icon.bmp</LogicalName>
</EmbeddedResource>
```

You can make your own icons using [GIMP](https://www.gimp.org/) by opening both of your default Icon.bmp and Icon.ico, edit them, and then export using the `Export As...` command (`CTRL + SHIFT + E`).

## Test

If you just want to test that it works, replace the default icons with these: [Icon.bmp](https://raw.githubusercontent.com/learn-monogame/learn-monogame.github.io/main/docs/how-to/custom-icon/Icon.bmp), [Icon.ico](https://raw.githubusercontent.com/learn-monogame/learn-monogame.github.io/main/docs/how-to/custom-icon/Icon.ico).

![Icon Preview](https://raw.githubusercontent.com/learn-monogame/learn-monogame.github.io/main/docs/how-to/custom-icon/Icon.bmp)

## Note

If you build your game on Linux or OSX, for example with an automated pipeline you will encounter a .NET Core bug where the icon doesn't get applied at all. You can read more about this issue in <https://github.com/dotnet/runtime/issues/3828>.
