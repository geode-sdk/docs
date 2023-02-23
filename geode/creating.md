# Creating a new Mod

These are instructions to create an empty mod.

## Prerequisites: 

 * [CMake](https://cmake.org/download/) (minimum v3.13.4)
 * A supported C++ compiler ([clang](https://releases.llvm.org/)/[MSVC](https://visualstudio.microsoft.com/downloads/))
 * [Geode CLI](https://github.com/geode-sdk/cli) (Make sure you [add the CLI](/info/installcli) to your path environment variable)
 * [git](https://git-scm.com/downloads) (Highly Recommended)

## Development Guide

1. Install [VS Code](https://code.visualstudio.com/)

2. Install the [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) and [CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools) extensions for VS Code

3. Open up the command line and navigate to any directory where you'd like to create the project

4. Run `geode new` and follow the prompts that follow

5. Open up the new directory in VS Code (hint: you can do this from the command line with `code <directory name>`)

6. Press F1 to open the Command Palette and run `CMake: Configure` (**make sure to select an x86/32-bit generator**, for example `Visual Studio Community 2019 Release - x86` or `Visual Studio Community 2022 Release - amd64_x86`)

7. Open up the Command Palette again and run `CMake: Select Variant` (select either `Release` or `RelWithDebInfo`)

Alternatively, you can run CMake manually without using VSCode or using another program (such as Sublime Text), however using the VSCode environment is highly recommended for Geode projects.

## Building With VSCode

To build with VSCode, simply press the `Build` button on the bottom bar or use the hotkey `F7`.

## Building manually

1. `cmake -B build -T host=x64 -A win32`

2. `cmake --build build --config Release`

