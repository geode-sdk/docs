# Installing Geode

## Using the installer

The official Geode installer is currently under heavy remodelling. It will be brought back here when it's ready :) For now, you will have to install the SDK manually.

## Manually (on Windows)

### Prerequisites

 * [CMake](https://cmake.org/download/) (minimum v3.13.4, prefer latest)
 * A supported C++ compiler ([clang](https://releases.llvm.org/)/[MSVC](https://visualstudio.microsoft.com/downloads/))
 * [git](https://git-scm.com/downloads)

1. Install Geode CLI (https://github.com/geode-sdk/cli/releases/latest)

  * Unzip the download file somewhere, and [add the directory with geode.exe to your user's PATH variable](https://helpdeskgeek.com/windows-10/add-windows-path-environment-variable/). You may need to restart your command line after adding the directory to your path.

2. Navigate somewhere on your computer using the command line and do `git clone https://github.com/geode-sdk/geode --recursive`

  * You may want to switch to the `ui` branch **if you're reading this document _before_ 2022/10/10**. Do this with `git checkout ui`

3. Build Geode yourself or ask HJfod for prebuilt binaries

4. [Create a new System Environment Variable](https://www.wikihow.com/Create-an-Environment-Variable-in-Windows-10) `GEODE_SDK` and make its value be the path to the `geode` repository on your computer (Example: `C:\Users\Steve\Documents\geode`)

5. All done :) See the [instructions for creating a mod](/docs/tutorial/creating.md) next
