---
title: 1.1. Required C++ Tools
order: 2
---

# Required C++ Tools
To be able to use the Geode SDK, you **will** need at least the following:
* [A C++ compiler](#compiler)
* [CMake](https://cmake.org/download/) - Version 3.21+ is required, and make sure to add to PATH when installing on windows.
* [git](https://git-scm.com/downloads) - Hey you. yes you. I know a lot of people skip this **but you will need it**. Don't come at us asking for why "could not find git for clone of json-populate"

## Compiler
To use the Geode SDK, and in turn make Geometry Dash mods, you will need either:
* [Visual Studio 2022+](#windows) on Windows
* [clang](#macos) on MacOS
* [A secret third thing](#linux) on Linux

### Windows
From the [Visual Studio](https://visualstudio.microsoft.com/downloads/) website, you can download the Visual Studio IDE along with the compiler. If you want **just** the compiler, look for *Build Tools for Visual Studio* further down in the page.

After launching the installer, look for **Desktop development with C++**. You may choose other features, but you **will** need at least ***MSVC*** and ***Windows SDK*** installed.

Once its installed, you should now have a working C++ compiler installed that is suited for GD mod development.

Please note that Visual Studio **2022** or higher is required. 2019 or lower will not work, as they don't support C++20 properly.

### MacOS

Install [brew](https://brew.sh/) if you don't already have it, and then run:
```bash
brew install llvm
```

### Linux
Linux is a bit more complicated, as obviously there's no linux release of GD (yet). Of course, you can run the Windows version of GD through software like [wine](https://www.winehq.org/) quite well, which is probably what you're already doing.

Because of that, this guide will set you up to [cross-compile](https://en.wikipedia.org/wiki/Cross_compiler) windows Geode mods from linux.

First, besides git and cmake, make sure you have `clang` and `lld` installed.

On Ubuntu:

```bash
apt install clang-17 clang-tools-17 lld-17
```

On Arch-based systems:

```bash
pacman -S clang lld
```

The next step will install the Windows SDK and a CMake toolchain. For ease of installation, first install [Geode CLI](/getting-started/geode-cli.md) and then come back here. If you want to do it manually, you can follow [this guide](https://gist.github.com/matcool/abb65ee59ded3766717c673014c3a2a7).

After installing the CLI, run this command to install all the needed tools:

```bash
geode sdk install-linux
```

Now you can proceed to [setting up the SDK](/getting-started/sdk.md).
