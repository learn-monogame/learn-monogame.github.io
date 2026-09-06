# Automate Release

In this article, we'll automate the release builds from a GitHub repository to an itch.io page. [itch.io](https://itch.io/) is an open marketplace for independent digital creators with a focus on independent video games.

Before you start, make sure you have an account on both [GitHub](https://github.com/join) and [itch.io](https://itch.io/register). Also make sure you have a GitHub repository with your MonoGame project along with a project page on itch.io.

## Steps

Here are the steps to setup a build pipeline for MonoGame that does a release on itch.io:

1. Install [Butler](https://itchio.itch.io/butler). Butler is a command-line tool to interact with itch.io.
2. Generate your API keys by running the `butler login` command.
3. You can then find your API key by going to <https://itch.io/user/settings/api-keys>.
    * Copy the API key that has `wharf` as the source.
4. Go to your GitHub repository `Settings`, then `Secrets and variables`, then `Actions`.
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
      windows-linux:

        runs-on: ubuntu-24.04

        env:
          MGFXC_WINE_PATH: /home/runner/.winemonogame

        steps:
        - uses: actions/checkout@v7
        - name: Setup dotnet
          uses: actions/setup-dotnet@v6
          with:
            dotnet-version: '10.0.x'
        - name: Get version from tag
          run: |
            TAGVERSION=$(git describe --tags --abbrev=0)
            echo "TAGVERSION=${TAGVERSION:1}" >> $GITHUB_ENV
        - name: Setup Wine
          run: |
            sudo add-apt-repository universe
            sudo apt update
            sudo apt install wget curl p7zip-full wine64
            wget -qO- https://monogame.net/downloads/net9_mgfxc_wine_setup.sh | bash
        - name: Build Windows
          run: dotnet publish ${{ env.PROJECT_PATH }} -r win-x64 -c Release --output artifacts/windows --self-contained -p:Version=${{ env.TAGVERSION }}
        - name: Build Linux
          run: dotnet publish ${{ env.PROJECT_PATH }} -r linux-x64 -c Release --output artifacts/linux --self-contained -p:Version=${{ env.TAGVERSION }}
        - name: Publish Windows build to itch.io
          uses: yeslayla/butler-publish-itchio-action@master
          env:
            BUTLER_CREDENTIALS: ${{ secrets.BUTLER_API_KEY }}
            CHANNEL: windows
            ITCH_GAME: ${{ env.ITCH_GAME_NAME }}
            ITCH_USER: ${{ env.ITCH_USER_NAME }}
            PACKAGE: artifacts/windows
            VERSION: ${{ env.TAGVERSION }}
        - name: Publish Linux build to itch.io
          uses: yeslayla/butler-publish-itchio-action@master
          env:
            BUTLER_CREDENTIALS: ${{ secrets.BUTLER_API_KEY }}
            CHANNEL: linux
            ITCH_GAME: ${{ env.ITCH_GAME_NAME }}
            ITCH_USER: ${{ env.ITCH_USER_NAME }}
            PACKAGE: artifacts/linux
            VERSION: ${{ env.TAGVERSION }}

      osx-build:

        runs-on: macos-15

        env:
          MGFXC_WINE_PATH: /Users/runner/.winemonogame

        steps:
        - uses: actions/checkout@v7
        - name: Setup dotnet
          uses: actions/setup-dotnet@v6
          with:
            dotnet-version: '10.0.x'
        - name: Get version from tag
          run: |
            TAGVERSION=$(git describe --tags --abbrev=0)
            echo "TAGVERSION=${TAGVERSION:1}" >> $GITHUB_ENV
        - name: Setup Wine
          run: |
            brew install wget p7zip curl
            brew install --cask wine-stable
            xattr -dr com.apple.quarantine "/Applications/Wine Stable.app"
            wget -qO- https://monogame.net/downloads/net9_mgfxc_wine_setup.sh | bash
        - name: Build Osx
          run: ./${{ env.PROJECT_PATH }}/package-osx.sh ${{ env.TAGVERSION }} artifacts/osx
        - name: Pack bundle for transport
          run: tar -czf osx.tar.gz -C artifacts/osx MyGame.app
        - uses: actions/upload-artifact@v7
          with:
            name: osx
            path: osx.tar.gz
            if-no-files-found: error
            retention-days: 1

      osx-publish:

        runs-on: ubuntu-24.04
        needs: osx-build

        steps:
        - name: Get version from tag
          run: echo "TAGVERSION=${GITHUB_REF_NAME:1}" >> $GITHUB_ENV
        - uses: actions/download-artifact@v8
          with:
            name: osx
        - name: Unpack bundle
          run: |
            mkdir -p artifacts/osx
            tar -xzf osx.tar.gz -C artifacts/osx
        - name: Publish OSX build to itch.io
          uses: yeslayla/butler-publish-itchio-action@master
          env:
            BUTLER_CREDENTIALS: ${{ secrets.BUTLER_API_KEY }}
            CHANNEL: osx
            ITCH_GAME: ${{ env.ITCH_GAME_NAME }}
            ITCH_USER: ${{ env.ITCH_USER_NAME }}
            PACKAGE: artifacts/osx
            VERSION: ${{ env.TAGVERSION }}
    ```
10. Replace line 9 to 11 with your own information.

    For example, a project URL for itch will look like: `[ITCH_USER_NAME].itch.io/[ITCH_GAME_NAME]`.

    `PROJECT_PATH` is the path to the directory where your `.csproj` is located. If it's located in the root of your repository, you can set the value to `~` (tilde): `PROJECT_PATH: ~`

## The macOS bundle

The Windows and Linux jobs publish a folder and push it as is. macOS needs an `.app` bundle instead, so it gets a script of its own. Save this as `package-osx.sh` next to your `.csproj` and make it executable with `chmod +x`:

```sh
#!/bin/sh
# Builds MyGame.app and leaves it in <output-dir>.
#
#     ./package-osx.sh 1.0.0 artifacts/osx

set -eu

VERSION=${1:?usage: package-osx.sh <version> <output-dir>}
OUTPUT=${2:?usage: package-osx.sh <version> <output-dir>}

PROJECT=$(cd "$(dirname "$0")" && pwd)

APP="$OUTPUT/MyGame.app"
CONTENTS="$APP/Contents"

rm -rf "$APP"
mkdir -p "$CONTENTS/MacOS" "$CONTENTS/Resources"

for arch in arm64 x64; do
    dotnet publish "$PROJECT" -c Release -r "osx-$arch" --self-contained \
        -p:Version="$VERSION" --output "$CONTENTS/MacOS/$arch"
done

# TitleContainer probes ../Resources then ../../Resources before falling back to
# the base directory, so both trees find one shared copy here.
mv "$CONTENTS/MacOS/arm64/Content" "$CONTENTS/Resources/Content"
rm -rf "$CONTENTS/MacOS/x64/Content"

cp "$PROJECT/Icon.icns" "$CONTENTS/Resources/Icon.icns"
sed "s/__VERSION__/$VERSION/g" "$PROJECT/Info.plist" > "$CONTENTS/Info.plist"
printf 'APPL????' > "$CONTENTS/PkgInfo"

cat > "$CONTENTS/MacOS/MyGame" <<'LAUNCHER'
#!/bin/sh
# exec replaces this process in place, so the game inherits the identity
# LaunchServices handed the bundle and the Dock tile stays put.
DIR=$(cd "$(dirname "$0")" && pwd)
case $(uname -m) in
    arm64) exec "$DIR/arm64/MyGame" "$@" ;;
    *)     exec "$DIR/x64/MyGame" "$@" ;;
esac
LAUNCHER
chmod +x "$CONTENTS/MacOS/MyGame" "$CONTENTS/MacOS/arm64/MyGame" "$CONTENTS/MacOS/x64/MyGame"

# Ad-hoc sign every Mach-O we ship. Most arrive signed already, though re-signing
# is idempotent and cheap next to shipping one stray image that gets the whole
# process killed on launch.
find "$CONTENTS/MacOS" -mindepth 2 -type f -exec sh -c '
    set -e
    for f do
        case $(file -b "$f") in
            *Mach-O*) codesign --force --sign - "$f" ;;
        esac
    done
' sh {} +

echo "Built $APP"
```

It reads an `Info.plist` sitting next to it. The script replaces `__VERSION__` with the tag version:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleDevelopmentRegion</key>
    <string>en</string>
    <key>CFBundleExecutable</key>
    <string>MyGame</string>
    <key>CFBundleIconFile</key>
    <string>Icon</string>
    <key>CFBundleIdentifier</key>
    <string>com.your-domain.MyGame</string>
    <key>CFBundleInfoDictionaryVersion</key>
    <string>6.0</string>
    <key>CFBundleName</key>
    <string>MyGame</string>
    <key>CFBundlePackageType</key>
    <string>APPL</string>
    <key>CFBundleShortVersionString</key>
    <string>__VERSION__</string>
    <key>CFBundleVersion</key>
    <string>__VERSION__</string>
    <key>LSApplicationCategoryType</key>
    <string>public.app-category.games</string>
    <key>LSMinimumSystemVersion</key>
    <string>12.0</string>
    <key>NSHighResolutionCapable</key>
    <true/>
    <key>NSPrincipalClass</key>
    <string>NSApplication</string>
</dict>
</plist>
```

You can build the `Icon.icns` from a 1024x1024 png on any Mac:

```sh
mkdir -p MyGame.iconset
for size in 16 32 128 256 512; do
    sips -z $size $size Icon.png --out MyGame.iconset/icon_${size}x${size}.png
    sips -z $((size * 2)) $((size * 2)) Icon.png --out MyGame.iconset/icon_${size}x${size}@2x.png
done
iconutil -c icns MyGame.iconset --output Icon.icns
```

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

To quickly check what the latest tag version is, you can use:
```
git describe --tags --abbrev=0
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
  ITCH_USER_NAME: apos
  ITCH_GAME_NAME: binaryinput
  PROJECT_PATH: Platforms/DesktopGL
```

---

Grabs the git tag, removes the `v` prefix and saves it as a GitHub Action environment variable.

```yml
TAGVERSION=$(git describe --tags --abbrev=0)
echo "TAGVERSION=${TAGVERSION:1}" >> $GITHUB_ENV
```

The `osx-publish` job never checks out the repository, so `git describe` has nothing to read there. It takes the tag out of `GITHUB_REF_NAME` instead.

```yml
run: echo "TAGVERSION=${GITHUB_REF_NAME:1}" >> $GITHUB_ENV
```

---

Setup [Wine](https://www.winehq.org/) for building shaders. Shader compilation goes through DirectX, so it doesn't run natively on Linux or macOS. Feel free to remove this if you don't have any shaders to speedup your builds considerably. It downloads Wine and configures it using [mgfxc_wine_setup.sh](https://github.com/MonoGame/MonoGame/blob/develop/Tools/MonoGame.Effect.Compiler/mgfxc_wine_setup.sh).

```yml
env:
  MGFXC_WINE_PATH: /home/runner/.winemonogame
```

```yml
sudo add-apt-repository universe
sudo apt update
sudo apt install wget curl p7zip-full wine64
wget -qO- https://monogame.net/downloads/net9_mgfxc_wine_setup.sh | bash
```

The home directory differs on the macOS runner, so `MGFXC_WINE_PATH` points at `/Users/runner/.winemonogame` there and Wine comes from brew.

---

Run the publish commands to get builds for each desktop platform using the `PROJECT_PATH` and `TAGVERSION` environment variables from earlier. You can remove or add the platforms you don't want. Each one gets a custom output folder. This is useful for knowing where the builds will be when it's time to upload to itch.io.

```yml
run: dotnet publish ${{ env.PROJECT_PATH }} -r win-x64 -c Release --output artifacts/windows --self-contained -p:Version=${{ env.TAGVERSION }}
run: dotnet publish ${{ env.PROJECT_PATH }} -r linux-x64 -c Release --output artifacts/linux --self-contained -p:Version=${{ env.TAGVERSION }}
```

---

macOS needs three things the other two platforms don't, which is why it takes two jobs of its own.

The bundle carries a full native tree per architecture rather than one merged universal tree. `lipo` can fuse the dylibs, though a self-contained .NET app also ships ReadyToRun framework assemblies that are compiled per architecture. `System.Private.CoreLib.dll` alone differs by more than a megabyte between the two, and those are managed PE files that `lipo` can't merge. So each tree stays exactly as `dotnet publish` produced it and the launcher script picks between them at startup.

Every Mach-O in the bundle then gets an ad-hoc signature. Apple Silicon kills any process whose images aren't signed, so an unsigned build is a build that never starts. Signing needs `codesign`, which only exists on macOS, and that's why `osx-build` runs on `macos-15`.

Getting the bundle back to a Linux runner takes some care. `upload-artifact` zips what you give it, and zip drops the executable bit off the launcher and both app hosts, so the bundle travels as a tar instead.

```yml
run: tar -czf osx.tar.gz -C artifacts/osx MyGame.app
```

The split into `osx-build` and `osx-publish` exists because `butler-publish-itchio-action` is a Docker action and GitHub only runs those on Linux.

The bundle itself stays unsigned on the outside. Without an Apple Developer ID there's nothing to notarize against, and a bundle seal that gets stripped in transit reads to Gatekeeper as tampering, which is worse than no seal at all.

---

The last step uses the [Butler Push](https://github.com/yeslayla/butler-publish-itchio-action) action feeding it the previous environment variables from earlier and the `BUTLER_API_KEY` API key added in step 6. This command gets run once for each platform to release on.

```yml
uses: yeslayla/butler-publish-itchio-action@master
env:
  BUTLER_CREDENTIALS: ${{ secrets.BUTLER_API_KEY }}
  CHANNEL: windows
  ITCH_GAME: ${{ env.ITCH_GAME_NAME }}
  ITCH_USER: ${{ env.ITCH_USER_NAME }}
  PACKAGE: artifacts/windows
  VERSION: ${{ env.TAGVERSION }}
```

## Read more

Read more about Butler: <https://itch.io/docs/butler/>.

Read more about GitHub Actions: <https://docs.github.com/en/free-pro-team@latest/actions>.

Visit <https://game.ci/docs/faq> to learn about more related projects.
