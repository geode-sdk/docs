---
title: 1.1. Required C++ Tools
order: 3
---

# Required C++ Tools
To be able to use the Geode SDK, you **will** need at least the following:
* [A C++ compiler](#compiler)
* [CMake](https://cmake.org/download/)
* [git](https://git-scm.com/downloads)

## Compiler
To use the Geode SDK, and in turn make Geometry Dash mods, you will need either:
* [Visual Studio](#windows) on Windows
* [clang](#macos) on MacOS
* [A secret third thing](#linux) on Linux

### Windows
From the [Visual Studio](https://visualstudio.microsoft.com/downloads/) website, you can download the Visual Studio IDE along with the compiler. If you want **just** the compiler, look for *Build Tools for Visual Studio* further down in the page.

After launching the installer, look for **Desktop development with C++**. You may choose other features, but you **will** need at least ***MSVC*** and ***Windows SDK*** installed.

Once its installed, you should now have a working C++ compiler installed that is suited for GD mod development.

### MacOS

Install [brew](https://brew.sh/) if you don't already have it, and then run:
```bash
brew install llvm
```

### Linux
Linux is well.. complicated