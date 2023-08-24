# Fixing VS Code Errors

Here are a couple of common errors people have experienced using Geode in VS Code and how to fix them.

## Include file `<Geode/Geode.hpp>` not found

![Image showing VS Code C++ extension unable to find Geode headers](/assets/include_path_error.png)

VS Code's C++ extension can't figure out the include path on its own. It is recommended to use the [CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools) extension, as it can automatically provide the includes through CMake, though it needs some setting up.

First, once you have the CMake Tools extension installed, open up your Geode mod, press **F1** and run `CMake: Select a Kit`. This will bring up a list of installed compilers on your machine.

![Image showing a bunch of compilers CMake detected in VS Code](/assets/win_compilers.png)

On Windows, you should go with a Visual Studio compiler that is either `x86` or `amd64_x86`. Note that there is also an `x86_amd64` compiler - you don't want to pick that one!

On Mac, you should go with Clang.

If you're on Windows, press **F1** again and run `CMake: Select Variant`.

![Image showing available build types on Windows: Debug, Release, MinSizeRel, and RelWithDebInfo](/assets/win_relwithdebinfo.png)

You should go with either `RelWithDebInfo` (recommended at least for development) or `Release`.

Now register CMake as the **Configuration Provider** for the C++ extension by running `C/C++: Edit Configurations (UI)` (you can also pick the JSON option if you know what you are doing).

![Image showing the "C/C++: Edit Configurations (UI)" command being run in VS Code](/assets/win_usecmake.png)

Scroll down to **Advanced** options, and set the Configuration Provider as `ms-vscode.cmake-tools`.

Now if you run `CMake: Configure`, assuming you have Geode properly installed, the include errors should go away.

If the errors still persist, try building the mod with `CMake: Build`. If the build is succesful but the errors don't disappear, try restarting VS Code.
