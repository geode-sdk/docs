---
layout: default
title: Building
parent: Installation
nav_order: 1
description: "Building instructions"
---

# Building

These are instructions for building the [Geode Loader](https://github.com/geode-sdk/loader). For building your own mods, see `A_PAGE_WE_HAVENT_MADE_YET_PLEASE_ADD_WHEN_ITS_MADE`.

### Prerequisites: 

 * [CMake](https://cmake.org/download/) (minimum v3.13.4)
 * A supported C++ compiler ([clang](https://releases.llvm.org/)/[MSVC](https://visualstudio.microsoft.com/downloads/))
 * [Geode CLI](https://github.com/geode-sdk/cli) (Highly Recommended)
 * [git](https://git-scm.com/downloads) (Highly Recommended)

### Quick instructions (for thigh-high programmers)

1. `git clone --recursive https://github.com/geode-sdk/loader.git`

2. `cmake -B build -T host=x64 -A win32`

3. `cmake --build build --config Release`

### Recommended way (for newbies)

1. Install [VS Code](https://code.visualstudio.com/)

2. Install the [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) and [CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools) extensions for VS Code

3. Open up the command line and navigate to any directory where you'd like to build the loader

4. `git clone https://github.com/geode-sdk/loader --recurse-submodules`

5. Open up the directory in VS Code

6. Press F1 to open the Command Palette and run `CMake: Configure` (make sure to select an x86 generator)

7. Open up the Command Palette again and run `CMake: Select Variant` (select either `Release` or `RelWithDebInfo`)

8. Click `Build` on the bottom status bar or run `CMake: Build`


