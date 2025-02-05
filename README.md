# dotnet-releases

A GitHub Action that automatically compiles and publishes .NET projects, then packages them and creates GitHub Releases for Windows, MacOS, and Linux.

```yml
name: Publish Release
on:
  release:
    types: [published]

jobs:
  release:
    name: Release
    strategy:
      matrix:
        kind: ['linux', 'windows', 'macOS']
        include:
          - kind: linux
            os: ubuntu-latest
            target: linux-x64
          - kind: windows
            os: windows-latest
            target: win-x64
          - kind: macOS
            os: macos-latest
            target: osx-x64
    runs-on: ${{ matrix.os }}
    steps:
      - name: Checkout
        uses: actions/checkout@v1

      - name: Setup dotnet
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: 5.0.202

      - name: Build
        shell: bash
        run: |
          tag=$(git describe --tags --abbrev=0)
          release_name="App-Hello-$tag-${{ matrix.target }}"
          # Build everything
          dotnet publish dotnet-releases.csproj --framework net5.0 --runtime "${{ matrix.target }}" -c Release -o "$release_name" --self-contained True -p:PublishSingleFile=True -p:IncludeNativeLibrariesForSelfExtract=true -p:PublishTrimmed=True -p:TrimMode=Link -p:DebugType=None -p:DebugSymbols=False
          # Pack files
          if [ "${{ matrix.target }}" == "win-x64" ]; then
            # Pack to zip for Windows
            7z a -tzip "${release_name}.zip" "./${release_name}/*"
          else
            tar czvf "${release_name}.tar.gz" "$release_name"
          fi
          # Delete output directory
          rm -r "$release_name"
      - name: Publish
        uses: softprops/action-gh-release@v0.1.14
        with:
          files: "App*"
```
