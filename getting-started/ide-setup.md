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

Some Visual Studio experience is recommended before you try to use this, but if you don't then you'll probably be fine.

Modern Visual Studio can handle CMake projects automatically, so assuming your VS has CMake support, just open your mod project folder. You'll know it's working if a console opens up at the bottom of your Visual Studio, and it starts gathering information from CMake.

Now, before you build, make sure to change these settings (these need to be changed in every single project you make):

1. Click the Debug options (This is usually a drop-down menu at the top of your screen that says "x64-Debug")
1. Click "Manage Configurations" inside that drop-down \
![Image showing the Manage Configurations button in the drop-down](/assets/vs_manage_configurations.png)

1. Change config type to Release or RelWithDebInfo. We recommend RelWithDebInfo, since it provides easier debugging. You **cannot** use Debug for this!
1. Change toolset to x86 (`msvc_x86`)
1. At this point you can also give your configuration a friendly name such as "default" or "release" or "mat" or something like that.
1. And make sure to press Ctrl+S to save your changes

Here's an example of a configuration that should work:
![Image showing a config that should work](/assets/vs_example_config.png)


Now you may build your mod, by pressing F7 or Ctrl+B (If those keybinds don't work, click Build at the top, then either Build or Build All)

If there are errors similar to VSCode (such as `#include <Geode/modify/MenuLayer.hpp> not found`) after you've built, restarting Visual Studio should make them go away.

If you get an error about Geode needing to be compiled for 32-bit, that means you didn't change your toolset to x86 above.

# Clion
No additional plugins are needed, the only thing you need to do is to set the CMake options correctly. When you open your mod's directory in CLion for the first time, you'll be met with an Open Project Wizard:
![Image showing the CLion Open Project Wizard](/assets/clion_openprojectwizard.png)

Here you need to make sure that:

1. Build type is set to Release or RelWithDebInfo, it **cannot** be Debug
2. Toolchain is set to *Visual Studio*
3. Generator is set to *Visual Studio 17 2022* or newer
4. Build directory is set to *build*
5. CMake options contains `-A win32`

In the end it should look like this:  
![Image showing the CLion Open Project Wizard set up for Geode](/assets/clion_openprojectwizardsetup.png)

Now you can click OK and CMake will run for the first time. The setup **will fail** until you build the project for the first time. \
In order to do that, go to the top right corner of the window where you'll see a dropdown saying *Add Configuration...*  
![Image showing the CLion Add Configuration dropdown](/assets/clion_addconfiguration.png)

Click it and choose *Edit Configurations...* and you'll be met with the Run/Debug Configurations window. Inside of it, click \
the plus button in the top left corner of the window and choose *CMake Application* from the popup:  
![Image showing the CLion Run/Debug Configurations window](/assets/clion_rundebugwindow.png)

In the created configuration, click the *Target* dropdown and choose *All Targets*. You can also change the name to \
something else than "Unnamed".  
![Image showing the CLion Run/Debug Configurations window with a configuration set up to compile the mod](/assets/clion_rundebugsetup.png)

Once that's done, you can click Apply and then OK. If it hasn't already, wait for CMake to finish its run. It might look \
something like this, with a line at the end saying `[Failed to reload]`  
![Image showing the failed CMake run output](/assets/clion_cmakerunfailed.png)

Now click the hammer icon in the top right corner next to build the mod for the first time - **make sure** that the dropdowns \
next to it say *RelWithDebInfo* and *Build Mod* (or however you called your build task in the step before):  
![Image showing the build button in CLion](/assets/clion_buildmod.png)

The build should end with a message saying `Build finished` (assuming you ran `geode config setup` before) and the empty mod \
should now be installed in the specified Geometry Dash instance. In order to make all the IDE features work correctly, \
reload CMake now, this run should end successfully with a message saying `[Finished]` at the end:  
![Image showing how to rerun CMake in CLion](/assets/clion_reloadcmake.png)

Don't forget to reload CMake after you add new files to the project.
