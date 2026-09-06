# MonoGame library

You have some code that you want to share between multiple projects? Making a library has never been easier.

## csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <PackageId>LibraryName</PackageId>
    <Description>Your Description</Description>
    <Authors>Your Name</Authors>
    <PackageTags>monogame;library</PackageTags>
    <PackageIcon>Icon.png</PackageIcon>
    <RepositoryUrl>https://github.com/learn-monogame/learn-monogame.github.io</RepositoryUrl>
    <PackageProjectUrl>https://github.com/learn-monogame/learn-monogame.github.io</PackageProjectUrl>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <PackageRequireLicenseAcceptance>false</PackageRequireLicenseAcceptance>
    <RepositoryType>git</RepositoryType>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <PublishRepositoryUrl>true</PublishRepositoryUrl>
    <EmbedUntrackedSources>true</EmbedUntrackedSources>
    <IncludeSymbols>true</IncludeSymbols>
    <SymbolPackageFormat>snupkg</SymbolPackageFormat>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="MonoGame.Framework.DesktopGL" PrivateAssets="All" Version="3.8.*" />
  </ItemGroup>

  <ItemGroup>
    <None Include="../Images/Icon.png" Pack="true" PackagePath="" />
  </ItemGroup>

  <PropertyGroup Condition="'$(GITHUB_ACTIONS)' == 'true'">
    <ContinuousIntegrationBuild>true</ContinuousIntegrationBuild>
  </PropertyGroup>

</Project>
```

MonoGame ships `net8.0`, so the library targets `net8.0` too. A `netstandard2.0` library restores against it with no compile assets at all, and nothing errors out until every MonoGame type you touch comes back as `CS0246`.

SourceLink comes from the .NET SDK, so `PublishRepositoryUrl` and `EmbedUntrackedSources` are all it needs.

## GitHub Action

In your repository's root, create the following directory structure: `.github/workflows/`.
In `workflows`, create a file called `release.yml`.

```yml
name: Release to NuGet

on:
  push:
    tags:
    - 'v*'

env:
  SOURCE: Source

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v7
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v6
      with:
        dotnet-version: '10.0.x'
    - name: Get version from tag
      run: |
        TAGVERSION=$(git describe --tags --abbrev=0)
        echo "TAGVERSION=${TAGVERSION:1}" >> $GITHUB_ENV
    - name: Pack with dotnet
      run: dotnet pack ${{ env.SOURCE }} -c Release -o ./artifacts -p:Version=${{ env.TAGVERSION }}
    - name: Push with dotnet
      run: dotnet nuget push ./artifacts/*.nupkg -k ${{ secrets.NuGetAPIKey }} -s https://api.nuget.org/v3/index.json --skip-duplicate
```
