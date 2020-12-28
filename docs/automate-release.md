# Automate Release

In this article, we'll automate the release builds from a GitHub repository to an itch.io page. <itch.io> is an open marketplace for independent digital creators with a focus on independent video games.

Before you start, make sure you have an account on both [GitHub](https://github.com/join) and [itch.io](https://itch.io/register). Also make sure you have a GitHub repository with your MonoGame project along with a project page on itch.io.

## Steps

Here are the steps to setup a build pipeline for MonoGame that does a release on itch.io:

1. Install [Butler](https://itchio.itch.io/butler). Butler is a command-line tool to interact with itch.io.
2. Generate your API keys by running the `butler login` command.
3. You can then find your API key by going to <https://itch.io/user/settings/api-keys>.
    * Copy the API key that has `wharf` as the source.
4. Go to your GitHub repository `Settings` and select the `Secrets` tab.
5. Create a `New repository secret`.
6. Name it `BUTLER_API_KEY` and paste the API key from step 4 as the value.
7. In your repository's root, create the following directory structure: `.github/workflows/`.
8. In `workflows`, create a file called `release.yml`.
9. Fill it with the following content:
    ```yml
    name: Release to itch.io

    on:
      push:
        tags:
        - 'v*'

    env:
      ITCH_USER_NAME: apos
      ITCH_GAME_NAME: binaryinput
      PROJECT_PATH: Platforms/DesktopGL

    jobs:
      build:

        runs-on: ubuntu-latest

        env:
          MGFXC_WINE_PATH: /home/runner/.winemonogame

        steps:
        - uses: actions/checkout@v2
        - name: Setup .NET Core
          uses: actions/setup-dotnet@v1
          with:
            dotnet-version: 3.1.101
        - name: Setup Wine
          run: |
            sudo dpkg --add-architecture i386
            sudo apt update
            wget -qO- https://dl.winehq.org/wine-builds/winehq.key | sudo apt-key add -
            sudo apt install software-properties-common
            sudo apt-add-repository 'deb http://dl.winehq.org/wine-builds/ubuntu/ bionic main'
            wget -qO- https://download.opensuse.org/repositories/Emulators:/Wine:/Debian/xUbuntu_18.04/Release.key | sudo apt-key add -
            sudo sh -c 'echo "deb https://download.opensuse.org/repositories/Emulators:/Wine:/Debian/xUbuntu_18.04/ ./" > /etc/apt/sources.list.d/obs.list'
            sudo apt update
            sudo apt-get install --install-recommends winehq-stable
            wget -qO- https://raw.githubusercontent.com/MonoGame/MonoGame/develop/Tools/MonoGame.Effect.Compiler/mgfxc_wine_setup.sh | sh
        - name: Build Windows
          run: dotnet publish ${{ env.PROJECT_PATH }} -r win-x64 -c Release --output build-windows
        - name: Build Osx
          run: dotnet publish ${{ env.PROJECT_PATH }} -r osx-x64 -c Release --output build-osx
        - name: Build Linux
          run: dotnet publish ${{ env.PROJECT_PATH }} -r linux-x64 -c Release --output build-linux
        - name: Publish Windows build to itch.io
          uses: josephbmanley/butler-publish-itchio-action@master
          env:
            BUTLER_CREDENTIALS: ${{ secrets.BUTLER_API_KEY }}
            CHANNEL: windows
            ITCH_GAME: ${{ env.ITCH_GAME_NAME }}
            ITCH_USER: ${{ env.ITCH_USER_NAME }}
            PACKAGE: build-windows
        - name: Publish OSX build to itch.io
          uses: josephbmanley/butler-publish-itchio-action@master
          env:
            BUTLER_CREDENTIALS: ${{ secrets.BUTLER_API_KEY }}
            CHANNEL: osx
            ITCH_GAME: ${{ env.ITCH_GAME_NAME }}
            ITCH_USER: ${{ env.ITCH_USER_NAME }}
            PACKAGE: build-osx
        - name: Publish Linux build to itch.io
          uses: josephbmanley/butler-publish-itchio-action@master
          env:
            BUTLER_CREDENTIALS: ${{ secrets.BUTLER_API_KEY }}
            CHANNEL: linux
            ITCH_GAME: ${{ env.ITCH_GAME_NAME }}
            ITCH_USER: ${{ env.ITCH_USER_NAME }}
            PACKAGE: build-linux
    ```
10. Replace line 9 to 11 with your own information.

    For example, a project URL for itch will look like: `[ITCH_USER_NAME].itch.io/[ITCH_GAME_NAME]`.

    `PROJECT_PATH` is the path to the directory where your `.csproj` is located. If it's located in the root of your repository, you can set the value to `~` (tilde): `PROJECT_PATH: ~`

## Trigger a release

With all that in place, you can do a release using the following git commands:

```
git tag v1
git push origin --tags
```

Increasing `v1` for each release. `v1`, `v2`, `v3`, etc.

## Explanation

Trigger a release when a new version tag is pushed.

```yml
on:
  push:
    tags:
    - 'v*'
```

---

Setup some environment variables for easier pipeline maintenance.

```yml
env:
  ITCH_GAME_NAME: BinaryInput
  ITCH_USER_NAME: apos
  BUILD_PATH: Platforms/DesktopGL
```

---

Setup [Wine](https://www.winehq.org/) for building shaders under Linux. Feel free to remove this if you don't have any shaders to speedup your builds considerably. It will download Wine and configure it using [mgfxc_wine_setup.sh](https://github.com/MonoGame/MonoGame/blob/develop/Tools/MonoGame.Effect.Compiler/mgfxc_wine_setup.sh).

```yml
env:
  MGFXC_WINE_PATH: /home/runner/.winemonogame
```

```yml
sudo dpkg --add-architecture i386
sudo apt update
wget -qO- https://dl.winehq.org/wine-builds/winehq.key | sudo apt-key add -
sudo apt install software-properties-common
sudo apt-add-repository 'deb http://dl.winehq.org/wine-builds/ubuntu/ bionic main'
wget -qO- https://download.opensuse.org/repositories/Emulators:/Wine:/Debian/xUbuntu_18.04/Release.key | sudo apt-key add -
sudo sh -c 'echo "deb https://download.opensuse.org/repositories/Emulators:/Wine:/Debian/xUbuntu_18.04/ ./" > /etc/apt/sources.list.d/obs.list'
sudo apt update
sudo apt-get install --install-recommends winehq-stable
wget -qO- https://raw.githubusercontent.com/MonoGame/MonoGame/develop/Tools/MonoGame.Effect.Compiler/mgfxc_wine_setup.sh | sh
```

---

Run the publish commands to get builds for each desktop platforms using the BUILD_PATH environment variable from earlier. You can remove or add the platforms you don't want. Each one gets a custom output folder. This is useful for knowing where the builds will be when it's time to upload to itch.io.

```yml
run: dotnet publish ${{ env.BUILD_PATH }} -r win-x64 -c Release --output build-windows
run: dotnet publish ${{ env.BUILD_PATH }} -r osx-x64 -c Release --output build-osx
run: dotnet publish ${{ env.BUILD_PATH }} -r linux-x64 -c Release --output build-linux
```

---

The last step uses the [Butler Push](https://github.com/josephbmanley/butler-publish-itchio-action) action feeding it the previous environment variables from earlier and the `BUTLER_API_KEY` API key added in step 6. This command gets run once for each platforms to release on.

```yml
uses: josephbmanley/butler-publish-itchio-action@master
env:
  BUTLER_CREDENTIALS: ${{ secrets.BUTLER_API_KEY }}
  ITCH_GAME: ${{ env.ITCH_GAME_NAME }}
  ITCH_USER: ${{ env.ITCH_USER_NAME }}
```

## Read more

Read more about Butler: <https://itch.io/docs/butler/>.

Read more about GitHub Actions: <https://docs.github.com/en/free-pro-team@latest/actions>.
