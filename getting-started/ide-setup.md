---
title: 3.1. IDE Setup
order: 6
---

# IDE Setup

You can use one of the following IDEs for developing Geode mods. Using an IDE is highly recommended, as they will give you code completion, intellisense, and simplify developing mods. Our recommended IDE is VS Code, which we also have a [dedicated extension](https://marketplace.visualstudio.com/items?itemName=GeodeSDK.geode) for.

* [Visual Studio Code](#visual-studio-code)
* [Visual Studio](#visual-studio)
* [CLion](#clion)

# Visual Studio Code

* [VSCode on Windows](#vscode-on-windows)
* [VSCode on Mac](#vscode-on-mac)
* [VSCode on Linux](#vscode-on-linux)

## VSCode on Windows

For VS Code, we recommend a few extensions:
 * [`C/C++`](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
 * [`CMake Tools`](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)
 * [`Geode`](https://marketplace.visualstudio.com/items?itemName=GeodeSDK.geode) - Not required, but will provide some nice features.

There are a few steps you should follow to get proper intellisense and code completion (you should only need to do these once per project, ideally):

1. With VS Code open on your project, press F1 and run `CMake: Select a Kit`. This will bring up a list of installed compilers on your machine.

![Image showing a bunch of compilers CMake detected in VS Code](/assets/win_compilers.png)

You should pick a **Visual Studio 2022+** compiler (specifically the `amd64` version), or Clang, but nothing else!

> :warning: Please pay attention to this

2. Now to select the build variant, press F1 and run `CMake: Select Variant`. We recommend **RelWithDebInfo** for easier debugging, as Debug may sometimes cause obscure crashes.

![Image showing available build types on Windows: Debug, Release, MinSizeRel, and RelWithDebInfo](/assets/win_relwithdebinfo.png)

3. Register CMake as the **Configuration Provider** for the C++ extension by pressing F1 and running `C/C++: Edit Configurations (UI)`. Scroll down to **Advanced** options, and set the Configuration Provider as `ms-vscode.cmake-tools`.

![Image showing the "C/C++: Edit Configurations (UI)" command being run in VS Code](/assets/win_usecmake.png)

Now, build your mod by pressing F1 and running `CMake: Build`. **You must build your mod first so that errors such as `#include <Geode/modify/MenuLayer.hpp> not found` go away.** If the mod was built successfully, the exit code at the end should be 0. If any errors still persist after building the mod, try restarting VS Code.

## VS Code on Mac

For VS Code on Mac, we recommend a few extensions:
 * [`clangd`](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd)
 * [`CMake Tools`](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)
 * [`Geode`](https://marketplace.visualstudio.com/items?itemName=GeodeSDK.geode) - Not required, but will provide some nice features.

There are a few steps you should follow to get proper intellisense (you should only need to do these once per project, ideally):

1. With VSCode open on your project, press F1 and run `CMake: Select a Kit`. You should choose **Clang**.

Now, build your mod by pressing F1 and running `CMake: Build`. **You must build your mod first so that errors such as `#include <Geode/modify/MenuLayer.hpp> not found` go away.** If the mod was built successfully, the exit code at the end should be 0. If any errors still persist after building the mod, try restarting VS Code.

## VSCode on Linux

The setup is very similar to [VSCode on Mac](#vscode-on-mac), so you can follow that. The primary difference is you have to manually create a kit, which can be done like so:

1. Open the Command Palette (with F1 or Ctrl + Shift + P) and run the command **CMake: Edit User-Local CMake Kits** (requires the CMake Tools extension and an opened CMake project)

2. Add a new kit to the list:

```json
{
    "name": "Geode Windows 64-bit",
    "compilers": {
        "C": "/usr/bin/clang",
        "CXX": "/usr/bin/clang++"
    },
    "isTrusted": true,
    "toolchainFile": "${userHome}/.local/share/Geode/cross-tools/clang-msvc-sdk/clang-msvc.cmake",
    "cmakeSettings": {
        "HOST_ARCH": "x86_64",
        "SPLAT_DIR": "${userHome}/.local/share/Geode/cross-tools/splat",
        "CMAKE_EXPORT_COMPILE_COMMANDS": 1
    }
}
```

If the toolchain and/or splat weren't installed via `geode sdk install-linux`, modify the paths accordingly.

Additionally, if you have Ninja installed, it's a good idea to set it as the generator instead of `make`, as it's much faster and uses multiple CPU cores by default unlike `make`. Add this to the JSON object above:

```json
"preferredGenerator": {
    "name": "Ninja"
}
```

Now, you should be able to select the kit and build mods!

Additionally, while this does not affect 90% of people, if you use SIMD intrinsics or include a library that does, you might get strange compile/link errors with the configuration above. This can be fixed by changing the compiler from `clang` to `clang-cl`, like so:

* `"toolchainFile"` to `${userHome}/.local/share/Geode/cross-tools/clang-msvc-sdk/clang-cl-msvc.cmake`
* `"compilers"` / `"C"` to `/usr/bin/clang-cl`
* `"compilers"` / `"CXX"` to `/usr/bin/clang-cl`

# Visual Studio

Some Visual Studio experience is recommended before you try to use this, but if you don't then you'll probably be fine.

Modern Visual Studio can handle CMake projects automatically, so assuming your VS has CMake support, just open your mod project folder. You'll know it's working if a console opens up at the bottom of your Visual Studio, and it starts gathering information from CMake.

Now, before you build, make sure to change these settings (these need to be changed in every single project you make):

1. Click the Debug options (This is usually a drop-down menu at the top of your screen that says "x64-Debug")
2. Click "Manage Configurations" inside that drop-down

![Image showing the Manage Configurations button in the drop-down](/assets/vs_manage_configurations.png)

3. Change config type to `Release` or `RelWithDebInfo`. We recommend `RelWithDebInfo`, since it provides easier debugging. **You cannot use `Debug` for this!**
4. Make sure the toolset is set to `x64`
5. At this point you can also give your configuration a friendly name such as "default" or "release" or something like that
6. And make sure to use Ctrl + S to save your changes

Here's an example of a configuration that should work:

![Image showing a config that should work](/assets/vs_example_config.png)

Now you may build your mod, by pressing F7 or Ctrl + B (If those keybinds don't work, click Build at the top, then either Build or Build All)

If there are errors similar to VS Code (such as `#include <Geode/modify/MenuLayer.hpp> not found`) after you've built, restarting Visual Studio should make them go away.

# CLion

* [CLion on Windows](#clion-on-windows)
* [CLion on Linux](#clion-on-linux)

## CLion on Windows

No additional plugins are needed - the only thing you need to do is to configure your toolchain correctly. When you open your mod's directory in CLion for the first time, you'll be met with an Open Project Wizard:

![Image showing the CLion Open Project Wizard](/assets/clion_openprojectwizard.png)

If this is the first time you're launching CLion, click the "Manage toolchains..." button, which will open a window where you can add a new toolchain.

Here's an example of how to set up Visual Studio toolchain:

![Image showing configured Visual Studio toolchain in CLion](/assets/clion_toolchainmsvc.png)

And here's an example of how to set up Clang toolchain:

![Image showing configured Clang toolchain in CLion](/assets/clion_toolchainllvm.png)

After that, in the Open Project Wizard, you need to make sure that:

1. Build type is set to `Release` or `RelWithDebInfo` (we recommend `RelWithDebInfo`, since `Debug` might cause obscure crashes)
2. Toolchain is set to *Visual Studio* or *Clang/LLVM*
3. Generator is set to *Visual Studio 17 2022* or *Ninja* (if you have it installed)

In the end it should look like this:  

![Image showing the CLion Open Project Wizard set up for Geode](/assets/clion_openprojectwizardsetup.png)

Now you can press OK and CMake will run for the first time. Wait until it finishes, and you should see a dropdown in the top right corner of the window that says `NameOfTheMod` (or whatever you called your project).
Click on it, and pick `NameOfTheMod_PACKAGE` instead, which will trigger post build steps and package/install the mod to Geometry Dash.

![Image showing the CLion Configuration dropdown](/assets/clion_addconfiguration.png)

Now click the hammer icon in the top right corner next to build the mod:

![Image showing the build button in CLion](/assets/clion_buildmod.png)

The build should end with a message saying `Build finished` and (assuming you ran `geode config setup` before) the mod should now be installed to Geometry Dash.

Don't forget to reload CMake after you add new files to the project.

![Image showing how to rerun CMake in CLion](/assets/clion_reloadcmake.png)

## CLion on Linux

Pretty much the same as on [Windows](#clion-on-windows) except for setting up the toolchain.

First step is to create an environment file with the following content:

```bash
export GEODE_SDK=$HOME/Documents/Geode
export SPLAT_DIR=$HOME/.local/share/Geode/cross-tools/splat
export CMAKE_TOOLCHAIN_FILE=$HOME/.local/share/Geode/cross-tools/clang-msvc-sdk/clang-msvc.cmake
export HOST_ARCH=x86_64
```

If the toolchain and/or splat weren't installed via `geode sdk install-linux`, modify the paths accordingly.

Save it as `geode-env.sh` in your home directory (or wherever you want, just remember the path).

> :warning: If you're using a non default shell (like fish), you might need to edit the environment file to use the correct syntax. For example, fish uses `set -gx VAR value` instead of `export VAR=value`.

Now, in the toolchain settings on the right side of the **Name** field, click the *Add environment* button and select the file you just created.

![Image showing how to configure crosscompiler toolchain in CLion](/assets/clion_toolchainlinux.png)

After that, you can follow the same steps as on Windows to build your mod.

For some specific mods/libraries (most people won't need this), you might also want to add the clang-cl toolchain. Everything is the same as with clang, except you have to use the `clang-cl-msvc.cmake` toolchain file instead. (Copy the environment file and change the `CMAKE_TOOLCHAIN_FILE` variable to point to the `clang-cl-msvc.cmake` file.)  
Also make sure you're using `clang-cl` as the compiler in the toolchain settings.