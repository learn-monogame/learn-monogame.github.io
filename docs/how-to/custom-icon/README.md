# Customize the game icon
It takes 2 seconds to set up a custom icon!

This guide is a follow up to [Get started](../get-started/README.md).

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
