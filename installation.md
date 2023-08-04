---
title: Installation
icon: download
description: Instructions for installing the Geode SDK & other developer tools
order: 0
---

# Installation

> :warning: This tutorial is for installing the Geode SDK **for developers**. If you are just looking to install the mod loader to use mods, see [our homepage](https://geode-sdk.org/install)

These are instructions for installing the **Geode SDK** and **CLI**.

### Prerequisites

To use Geode, you need at least the following installed on your computer:

 * [CMake](https://cmake.org/download/) (minimum v3.13.4, prefer latest)
 * A supported C++ compiler ([MSVC](https://visualstudio.microsoft.com/downloads/) for Windows, [clang](https://releases.llvm.org/) for MacOS. If you use Visual Studio, make sure to select the `Desktop development with C++` kit from the installer)
 * [git](https://git-scm.com/downloads)

## Installing with CLI

The following steps guide you through installing the Geode CLI and SDK. If you would like to manually build them from source, [see below](#building-from-source).

Geode CLI is the command-line interface for working with Geode. It handles packaging mods into `.geode` files, creating spritesheets, creating bitmap fonts, etc. The CLI is required for compiling the Geode loader, and is extremely helpful for working with your own mods, as it automatically packages all the resources into a `.geode` file and installs it to GD.

You can also use the CLI for installing the Geode SDK, which is going to be used in this tutorial.

1. Download the [latest release of CLI](https://github.com/geode-sdk/cli/releases/latest)

2. Unzip the downloaded file in some directory on your computer
 * On Windows, you also need to [add the executable to your PATH](/geode/cli-to-path.md).

3. Install the Geode Mod Loader to GD (see [our homepage](https://geode-sdk.org/install)) 

4. Open a terminal and run `geode sdk install` to install the SDK (You can provide an argument to `geode sdk install <path>` if you want to install somewhere other than the default path). If the command wasn't found, [make sure CLI is in your PATH](/geode/cli-to-path.md), and try restarting your computer.
    * The installation should automatically set the `GEODE_SDK` environment variable to point to the SDK on Windows. You might need to restart your terminal or IDE, or even have to restart your computer. If it still doesn't work then you'll have to [set it manually](/geode/sdk-env-var.md), as it is required.

5. Run `geode config setup` from the command line to set up a profile in CLI

5. Run `geode sdk install-binaries` to install prebuilt binaries, or [build Geode yourself](#building-from-source)
    * You can pick whether you want the `nightly` or `stable` branch of Geode. `nightly` is the latest commit of the `main` branch, and contains all of the latest and greatest features but may also feature crashes and possibly not even compile, whereas `stable` is the latest released version. Use `geode sdk update nightly` or `geode sdk update stable` to switch between the two branches.

6. All done :) See the [Getting started](/getting-started.md) for information on how to start creating mods with Geode.

## Recommended developer tools

The CLI and SDK are strictly the only things needed to develope Geode mods, however here is also a few recommended developer tools to make your modding experience better:

 * [DevTools](https://github.com/geode-sdk/devtools) - In-game developer tools mod that lets you inspect the current scene
 * [The Geode VS Code extension](https://marketplace.visualstudio.com/items?itemName=GeodeSDK.geode) - Adds lots of useful little tools such a sprite browser to VS Code

## Building from Source

Building Geode from source has the same prerequisites as before, aswell as requiring CLI to be installed. If you'd like to build Geode CLI from source, you also need to install [Rust](https://rustup.rs/). Building the CLI is as simple as running `cargo build`.

### Quick instructions (for thigh-high programmers)

1. `git clone --recursive https://github.com/geode-sdk/geode.git`

2. `cmake -B build -T host=x64 -A win32`

3. `cmake --build build --config Release`

### Recommended way (for normal people)

1. Install [VS Code](https://code.visualstudio.com/)

2. Install the [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) and [CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools) extensions for VS Code

3. Open up the command line and navigate to any directory where you'd like to build the loader

4. `git clone https://github.com/geode-sdk/geode`

5. Open up the directory in VS Code (tip: you can do this from the command line via `code ./geode`)

6. Press F1 to open the Command Palette and run `CMake: Configure`. On Windows, make sure to select an x86 generator, for example `Visual Studio Community 2019 Release - amd64_x86`. Note that for Visual Studio generators, the second number is the target architecture, which must be x86.

7. Open up the Command Palette again and run `CMake: Select Variant` (select either `Release` or `RelWithDebInfo`). This may also be done through the bottom status bar. There should be a button that displays `Debug`. Click on this and change it to the option in the dropdown that displays `Release`

8. Click `Build` on the bottom status bar or run `CMake: Build`

9. If building was succesful, you can move the files from `bin/nightly` to your Geometry Dash folder (move the contents of resources to `<GD folder>/geode/resources/geode.loader`)
