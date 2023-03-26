# Creating a new Mod

These are instructions to create an empty mod.

## Prerequisites: 

 * [CMake](https://cmake.org/download/) (minimum v3.21)
 * A supported C++ compiler ([clang](https://releases.llvm.org/) on MacOS, [MSVC](https://visualstudio.microsoft.com/downloads/) on Windows)
 * [Geode CLI](https://github.com/geode-sdk/cli) (**Make sure you [add the CLI](/geode/installcli) to your path environment variable**)
 * [git](https://git-scm.com/downloads)

## Creating the mod

You can easily create a mod using the [Geode CLI](/geode/installcli), by simply running this command in a terminal / command prompt and answering the prompts that follow.
```
geode new
```

This will clone a template mod which you can then start editing to do whatever you want.

## Building

To build a mod, you can either do it [manually](#manually) or [with Visual Studio Code](#recommended-setup-using-vs-code).

**IMPORTANT**: You will not be able to build a mod if you do not have [set up the SDK for developers](/installation). \
Make sure the `GEODE_SDK` enviroment variable works.

### Manually

```bash
# Configure (for Windows)
cmake -B build -A win32
# Configure (for MacOS)
cmake -B build

# And then build
cmake --build build --config Release
```

### Recommended Setup using VS Code

1. Make sure you have the [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) and [CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools) extensions for VS Code

2. Open up the directory of your mod in VS Code (hint: you can do this from the command line with `code <directory name>`)

3. Press F1 to open the Command Palette and run `CMake: Configure`  
  3.1 If you're on **Windows** make sure to select an x86/32-bit generator, for example
    - `Visual Studio Community 2022 Release - x86`
    - `Visual Studio Community 2022 Release - amd64_x86`

4. Open up the Command Palette again and run `CMake: Select Variant` (select either `Release` or `RelWithDebInfo`)

If you have the CMake Tools extension, you can simply press the `Build` button on the bottom bar or use the hotkey `F7`.
