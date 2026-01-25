---
title: 1.1. Required C++ Tools
order: 2
---

# Required C++ Tools

To be able to use the Geode SDK, you **will** need at least the following:
 * [A C++ compiler](#compiler)
 * [CMake](https://cmake.org/download/) - Version 3.29+ is required - make sure to add to PATH when installing on Windows.
 * [Git](https://git-scm.com/downloads) - Hey you. Yes, you! I know a lot of people skip this step **but you will need it**. Don't come at us asking for why you "could not find git for clone of json-populate".

## Compiler

To use the Geode SDK, and in turn make Geometry Dash mods, you will need either:
 * [Clang](#windows-clang) (recommended) or [Visual Studio 2022+](#windows-visual-studio) on Windows
 * [Clang](#macos) on MacOS
 * [A secret third thing](#linux) on Linux

### Windows (Clang)

First, install [LLVM](https://github.com/llvm/llvm-project) and [Ninja](https://github.com/ninja-build/ninja). The easiest way is using Scoop:
```
scoop install llvm ninja
```

LLVM itself does not come with Windows SDK and CRT libraries, so you will need a minimal Visual Studio install for these, which is described in the next section.

### Windows (Visual Studio)

Please note that Visual Studio **2022** or higher is required. If you have an older version already installed, you should upgrade to the latest available.

Unless you want to install Visual Studio (the editor) itself, we recommend installing just the build tools. Open the VS [download page](https://visualstudio.microsoft.com/downloads), scroll to the bottom, and under "Tools for Visual Studio" download **Build Tools for Visual Studio**.

After launching the installer, select **Desktop development with C++** and optionally deselect all features except for **MSVC Build Tools** and **Windows SDK**, like on the screenshot below. Click Install and wait for it to finish.

![Image showing VS installer](/assets/vs_installer.png)

Once Visual Studio is installed, you should now have a working C++ compiler that is suited for GD mod development.

### MacOS

Install [brew](https://brew.sh/) if you don't already have it, and then run:
```bash
brew install llvm
```

### Linux

Linux is a bit more complicated, as there's no official Linux release of GD (yet). Of course, you can run the Windows version of GD through software like [wine](https://www.winehq.org/) quite well, which is probably what you're already doing.

Because of that, this guide will set you up to [cross-compile](https://en.wikipedia.org/wiki/Cross_compiler) Windows Geode mods from Linux.

First, besides Git and CMake, make sure you have `clang` and `lld` installed.

For CMake make sure you have at least version 3.29, some distributions like Debian and Linux Mint ship older versions which are not able to cross-compile Geode mods.

On Ubuntu:

```bash
apt install clang-19 clang-tools-19 lld-19
```

On Arch-based systems:

```bash
pacman -S clang lld llvm
```

The next step will install the Windows SDK and a CMake toolchain. For ease of installation, first install [the Geode CLI](/getting-started/geode-cli.md) and then come back here. If you want to do it manually, you can follow [this guide](https://gist.github.com/matcool/abb65ee59ded3766717c673014c3a2a7).

After installing the CLI, run this command to install all the needed tools:

```bash
geode sdk install-linux
```

Now you can proceed to [setting up Geode CLI](/getting-started/geode-cli.md).
