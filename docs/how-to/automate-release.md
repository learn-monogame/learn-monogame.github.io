# Automate Release

In this article, we'll automate the release builds from a GitHub repository to an itch.io page. [itch.io](https://itch.io/) is an open marketplace for independent digital creators with a focus on independent video games.

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

        runs-on: ubuntu-20.04

        env:
          MGFXC_WINE_PATH: /home/runner/.winemonogame

        steps:
        - uses: actions/checkout@v2
        - name: Setup dotnet
          uses: actions/setup-dotnet@v3
          with:
            dotnet-version: '8.0.x'
        - name: Get version from tag
          run: |
            TAGVERSION=$(git describe --tags --abbrev=0)
            echo "TAGVERSION=${TAGVERSION:1}" >> $GITHUB_ENV
        - name: Setup Wine
          run: |
            sudo apt update
            sudo apt install wine64 p7zip-full
            wget -qO- https://raw.githubusercontent.com/MonoGame/MonoGame/master/Tools/MonoGame.Effect.Compiler/mgfxc_wine_setup.sh | sh
        - name: Build Windows
          run: dotnet publish ${{ env.PROJECT_PATH }} -r win-x64 -c Release --self-contained --output artifacts/windows
        - name: Build Osx
          run: dotnet publish ${{ env.PROJECT_PATH }} -r osx-x64 -c Release --self-contained --output artifacts/osx
        - name: Build Linux
          run: dotnet publish ${{ env.PROJECT_PATH }} -r linux-x64 -c Release --self-contained --output artifacts/linux
        - name: Publish Windows build to itch.io
          uses: josephbmanley/butler-publish-itchio-action@master
          env:
            BUTLER_CREDENTIALS: ${{ secrets.BUTLER_API_KEY }}
            CHANNEL: windows
            ITCH_GAME: ${{ env.ITCH_GAME_NAME }}
            ITCH_USER: ${{ env.ITCH_USER_NAME }}
            PACKAGE: artifacts/windows
            VERSION: ${{ env.TAGVERSION }}
        - name: Publish OSX build to itch.io
          uses: josephbmanley/butler-publish-itchio-action@master
          env:
            BUTLER_CREDENTIALS: ${{ secrets.BUTLER_API_KEY }}
            CHANNEL: osx
            ITCH_GAME: ${{ env.ITCH_GAME_NAME }}
            ITCH_USER: ${{ env.ITCH_USER_NAME }}
            PACKAGE: artifacts/osx
            VERSION: ${{ env.TAGVERSION }}
        - name: Publish Linux build to itch.io
          uses: josephbmanley/butler-publish-itchio-action@master
          env:
            BUTLER_CREDENTIALS: ${{ secrets.BUTLER_API_KEY }}
            CHANNEL: linux
            ITCH_GAME: ${{ env.ITCH_GAME_NAME }}
            ITCH_USER: ${{ env.ITCH_USER_NAME }}
            PACKAGE: artifacts/linux
            VERSION: ${{ env.TAGVERSION }}
    ```
10. Replace line 9 to 11 with your own information.

    For example, a project URL for itch will look like: `[ITCH_USER_NAME].itch.io/[ITCH_GAME_NAME]`.

    `PROJECT_PATH` is the path to the directory where your `.csproj` is located. If it's located in the root of your repository, you can set the value to `~` (tilde): `PROJECT_PATH: ~`

## Trigger a release

With all that in place, you can do a release using the following git commands:

```
git tag v1
git push origin tag v1
```

Increasing `v1` for each release. `v1`, `v2`, `v3`, etc. You can also use [Semantic Versioning](https://semver.org/) if you want.

If you make a mistake, or the pipeline fails for some reason you can delete the tag with:

```
git tag --delete v1
git push origin tag --delete v1
```

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

Grabs the git tag, removes the `v` prefix and saves it as a GitHub Action environment variable.

```yml
TAGVERSION=$(git describe --tags --abbrev=0)
echo "TAGVERSION=${TAGVERSION:1}" >> $GITHUB_ENV
```

---

Setup [Wine](https://www.winehq.org/) for building shaders under Linux. Feel free to remove this if you don't have any shaders to speedup your builds considerably. It will download Wine and configure it using [mgfxc_wine_setup.sh](https://github.com/MonoGame/MonoGame/blob/master/Tools/MonoGame.Effect.Compiler/mgfxc_wine_setup.sh).

```yml
env:
  MGFXC_WINE_PATH: /home/runner/.winemonogame
```

```yml
sudo apt update
sudo apt install wine64 p7zip-full
wget -qO- https://raw.githubusercontent.com/MonoGame/MonoGame/master/Tools/MonoGame.Effect.Compiler/mgfxc_wine_setup.sh | sh
```

---

Run the publish commands to get builds for each desktop platforms using the BUILD_PATH environment variable from earlier. You can remove or add the platforms you don't want. Each one gets a custom output folder. This is useful for knowing where the builds will be when it's time to upload to itch.io.

```yml
run: dotnet publish ${{ env.BUILD_PATH }} -r win-x64 -c Release --self-contained --output artifacts/windows
run: dotnet publish ${{ env.BUILD_PATH }} -r osx-x64 -c Release --self-contained --output artifacts/osx
run: dotnet publish ${{ env.BUILD_PATH }} -r linux-x64 -c Release --self-contained --output artifacts/linux
```

---

The last step uses the [Butler Push](https://github.com/josephbmanley/butler-publish-itchio-action) action feeding it the previous environment variables from earlier and the `BUTLER_API_KEY` API key added in step 6. This command gets run once for each platforms to release on.

```yml
uses: josephbmanley/butler-publish-itchio-action@master
env:
  BUTLER_CREDENTIALS: ${{ secrets.BUTLER_API_KEY }}
  ITCH_GAME: ${{ env.ITCH_GAME_NAME }}
  ITCH_USER: ${{ env.ITCH_USER_NAME }}
  VERSION: ${{ env.TAGVERSION }}
```

## Read more

Read more about Butler: <https://itch.io/docs/butler/>.

Read more about GitHub Actions: <https://docs.github.com/en/free-pro-team@latest/actions>.

Visit <https://game.ci/docs/faq> to learn about more related projects.
