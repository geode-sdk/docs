---
title: 1.1. Required C++ Tools
order: 2
---

# Required C++ Tools

To be able to use the Geode SDK, you **will** need at least the following:
 * [A C++ compiler](#compiler)
 * [CMake](https://cmake.org/resources/) - Version 3.29+ is required - make sure to add to PATH when installing on Windows.
 * [Git](https://git-scm.com/install/) - Hey you. Yes, you! I know a lot of people skip this step **but you will need it**. Don't come at us asking for why you "could not find git for clone of json-populate".

## Compiler

To use the Geode SDK, and in turn make Geometry Dash mods, you will need either:
 * [Clang](#windows) on Windows
 * [Clang](#macos) on MacOS
 * [A secret third thing](#linux) on Linux

### Windows

First, install [LLVM](https://github.com/llvm/llvm-project) and [Ninja](https://github.com/ninja-build/ninja). The easiest way is using Scoop:
```
scoop install llvm ninja
```

LLVM itself does not come with Windows SDK and CRT libraries, so you will need a minimal Visual Studio install for these.

Unless you want to install Visual Studio (the editor) itself, we recommend installing just the build tools. Open the VS [download page](https://visualstudio.microsoft.com/downloads/), scroll to the bottom, and under "Tools for Visual Studio" download **Build Tools for Visual Studio**.

Please note that Visual Studio **2022** or higher is required. If you have an older version already installed, you should upgrade to the latest available.

After launching the installer, select **Desktop development with C++** and optionally deselect all features except for **MSVC Build Tools** and **Windows SDK**, like on the screenshot below. Click Install and wait for it to finish.

![Image showing VS installer](/assets/vs_installer.png)

Once Visual Studio is installed, you should now have a working C++ compiler that is suited for GD mod development.

Note that although it is possible to use MSVC from Visual Studio on its own to compile Geode mods, we **do not** recommend using it, since you may not be able to compile a mod using it (MSVC likes to run into segfaults while compiling certain libraries), and it is also slower overall.

### MacOS

Install [brew](https://brew.sh/) if you don't already have it, and then run:
```bash
brew install llvm
```

### Linux

Linux is a bit more complicated, as there's no official Linux release of GD (yet). Of course, you can run the Windows version of GD through software like [wine](https://www.winehq.org/) quite well, which is probably what you're already doing.

Because of that, this guide will set you up to [cross-compile](https://en.wikipedia.org/wiki/Cross_compiler) Windows Geode mods from Linux.

First, besides Git and CMake, make sure you have `clang` and `lld` installed.

For CMake make sure you have at least version 3.29, some distributions like Debian 12 and Linux Mint ship older versions which are not able to cross-compile Geode mods.

On Debian-based systems (Ubuntu, Linux Mint, ...):

```bash
apt install clang clang-tools lld llvm cmake git
```
Note: Some LTS distributions (Linux Mint) may offer old versions of Clang and CMake by default. As of the writing of this page, Clang 19 (available by appending `-19` to every package name except cmake and git on outdated distros) is the absolute minimum usable for compiling Geode mods, though version 21 or higher are recommended and compatiblity with Clang 19 may not be maintained. For updating CMake you may use the official [Kitware APT Repository](https://apt.kitware.com/).

On Fedora:
```bash
dnf install clang lld llvm cmake git
```

On Arch-based systems:

```bash
pacman -S clang lld llvm cmake git
```

Alternatively, you can use [brew](https://brew.sh/) (Linuxbrew) on Linux just like on macOS:

```bash
brew install llvm
```

The next step will install the Windows SDK and a CMake toolchain. For ease of installation, first install [the Geode CLI](/getting-started/geode-cli.md) and then come back here. If you want to do it manually, you can follow [this guide](https://gist.github.com/matcool/abb65ee59ded3766717c673014c3a2a7).

After installing the CLI, run this command to install all the needed tools:

```bash
geode sdk install-linux
```

Now you can proceed to [setting up Geode CLI](/getting-started/geode-cli).
