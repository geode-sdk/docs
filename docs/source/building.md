# Building

These are instructions for building the [Geode Loader](https://github.com/geode-sdk/geode). For building your own mods, see [Getting Started](/docs/starting).

### Prerequisites: 

 * [CMake](https://cmake.org/download/) (minimum v3.13.4)
 * A supported C++ compiler ([clang](https://releases.llvm.org/)/[MSVC](https://visualstudio.microsoft.com/downloads/))
 * [Geode CLI](https://github.com/geode-sdk/cli)
 * [git](https://git-scm.com/downloads)

### Quick instructions (for thigh-high programmers)

1. `git clone --recursive https://github.com/geode-sdk/geode.git`

2. `cmake -B build -T host=x64 -A win32`

3. `cmake --build build --config Release`

### Recommended way (for normal people)

1. Install [VS Code](https://code.visualstudio.com/)

2. Install the [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) and [CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools) extensions for VS Code

3. Open up the command line and navigate to any directory where you'd like to build the loader

4. `git clone https://github.com/geode-sdk/geode --recursive`

5. Open up the directory in VS Code

6. Press F1 to open the Command Palette and run `CMake: Configure` (make sure to select an x86 generator, for example `Visual Studio Community 2019 Release - amd64_x86`. Note that for Visual Studio generators, the second number is the target architecture, which must be x86.)

7. Open up the Command Palette again and run `CMake: Select Variant` (select either `Release` or `RelWithDebInfo`). This may also be done through the bottom status bar. There should be a button that displays `Debug`. Click on this and change it to the option in the dropdown that displays `Release`

8. Click `Build` on the bottom status bar or run `CMake: Build`

9. If building was succesful, you can move the files from `bin/nightly` to your Geometry Dash folder (move the contents of resources to `<GD folder>/geode/resources/geode.loader`)


