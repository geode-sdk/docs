---
title: 3.1. IDE Setup
order: 6
---

# IDE Setup

If youre using some IDE you may want to do a few things for your Geode projects.

* [Visual Studio Code](#visual-studio-code)
* [Visual Studio](#visual-studio)
* [Clion](#clion)

# Visual Studio Code

* [VSCode on Windows](#vscode-on-windows)
* [VSCode on Mac](#vscode-on-mac)

## VSCode on Windows

For VSCode, we recommend a few extensions:
- `C/C++`
- `CMake Tools`
- `Geode` - Not required, but will provide some nice features.

There are a few steps you should follow to get proper intellisense (you should only need to do these once per project, ideally):


1. With VSCode open on your project, **press F1** and run `CMake: Select a Kit`. This will bring up a list of installed compilers on your machine.
![Image showing a bunch of compilers CMake detected in VS Code](/assets/win_compilers.png) \
\
You should pick a **Visual Studio** compiler, using either the `x86` or `amd64_x86` version. Nothing else!!

> :warning: Please pay attention to this
2. Now select the build variant, **press F1** and run `CMake: Select Variant`. \
**YOU MUST PICK ANYTHING BUT DEBUG** or your mod will not compile. \
We recommend **RelWithDebInfo** for easier debugging. \
![Image showing available build types on Windows: Debug, Release, MinSizeRel, and RelWithDebInfo](/assets/win_relwithdebinfo.png)

3. Register CMake as the **Configuration Provider** for the C++ extension by **pressing F1** and running `C/C++: Edit Configurations (UI)`:\
Scroll down to **Advanced** options, and set the Configuration Provider as `ms-vscode.cmake-tools`. \
![Image showing the "C/C++: Edit Configurations (UI)" command being run in VS Code](/assets/win_usecmake.png)

Now, **build your mod** by **pressing F1** and running `CMake: Build`. \
You **must** build your mod first so that errors such as `#include <Geode/modify/MenuLayer.hpp> not found` go away.

Make sure your mod built successfully, the exit code at the end should be 0.

If the errors still persist, try restarting VS Code.

## VSCode on Mac

For VSCode, we recommend a few extensions:
- `clangd`
- `CMake Tools`
- `Geode` - Not required, but will provide some nice features.

There are a few steps you should follow to get proper intellisense (you should only need to do these once per project, ideally):

1. With VSCode open on your project, **press F1** and run `CMake: Select a Kit`. You should choose **clang**

Now, **build your mod** by **pressing F1** and running `CMake: Build`. \
You **must** build your mod first so that errors such as `#include <Geode/modify/MenuLayer.hpp> not found` go away.

Make sure your mod built successfully, the exit code at the end should be 0.

If the errors still persist, try restarting VS Code.

# Visual Studio
TODO
# Clion
TODO