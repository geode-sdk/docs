---
title: 3.1. IDE Setup
order: 6
---

# IDE Setup

If you're using some IDE you may want to do a few things for your Geode projects.

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

Modern Visual Studio can handle CMake projects, so assuming your VS has CMake support, just open your mod project folder.

Now, before you build, make sure to change these settings:

1. Click the Debug options
2. Manage Configurations
3. Change config type to Release or RelWithDebInfo. You **cannot** use Debug for this.
4. Change toolset to x86

Now you may build your mod! If your mod successfully built (you should see some info about your .geode file being installed) and you still have errors, **restart Visual Studio** to make them go away.

# CLion

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
