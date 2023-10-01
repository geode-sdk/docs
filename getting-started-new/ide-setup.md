---
title: 3.1. Creating a new mod
order: 6
---

# IDE Setup

If youre using some IDE you may want to do a few things for your Geode projects.

* [Visual Studio Code](#visual-studio-code)
* [Visual Studio](#visual-studio)
* [Clion](#clion)

# Visual Studio Code

For VSCode, we recommend a few extensions:
- `C/C++` on Windows
- `clangd` on MacOS
- `CMake Tools`
- `Geode`

None of these extensions are "required", but they will make your experience much better.

There are a few steps you should follow to get proper intellisense (you should only need to do these once per project, ideally):

1. With VSCode open on your project, press **F1** and run `CMake: Select a Kit`. This will bring up a list of installed compilers on your machine.
![Image showing a bunch of compilers CMake detected in VS Code](/assets/win_compilers.png)
On **Windows**, you should go with a Visual Studio compiler that is either `x86` or `amd64_x86`. Note that there is also an `x86_amd64` compiler - you don't want to pick that one!\
On **Mac**, you should go with Clang.

    1. If you're on **Windows**, press **F1** again and run `CMake: Select Variant`.
![Image showing available build types on Windows: Debug, Release, MinSizeRel, and RelWithDebInfo](/assets/win_relwithdebinfo.png)\
You should go with either `RelWithDebInfo` (recommended at least for development) or `Release`.

2. On **Windows**, register CMake as the **Configuration Provider** for the C++ extension by running `C/C++: Edit Configurations (UI)`.\
\
Scroll down to **Advanced** options, and set the Configuration Provider as `ms-vscode.cmake-tools`.
![Image showing the "C/C++: Edit Configurations (UI)" command being run in VS Code](/assets/win_usecmake.png)

Now if you run `CMake: Configure`, assuming you have Geode properly installed, the include errors should go away.

If the errors still persist, try building the mod with `CMake: Build`. If the build is succesful but the errors don't disappear, try restarting VS Code.

# Visual Studio
TODO
# Clion
TODO