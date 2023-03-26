---
title: Installation
icon: download
description: Instructions for installing Geode
order: 0
---

# Installing Geode

## For Users

> :warning: Please note that Geode is **incompatible with all other mod loaders and mods**. We're porting stuff like Mega Hack and BetterEdit over, but don't expect their old releases to work.

Geode is installed using **its own installer**. You can find the latest version [on the Geode homepage](https://geode-sdk.org) or [on GitHub](https://github.com/geode-sdk/installer/releases/latest).

Geode is currently available on **Windows** and **MacOS**; Android and iOS support is planned in the future.

## Installing Geode SDK (for Developers)

### Prerequisites

 * [CMake](https://cmake.org/download/) (minimum v3.13.4, prefer latest)
 * A supported C++ compiler ([MSVC](https://visualstudio.microsoft.com/downloads/) for Windows, [clang](https://releases.llvm.org/) for MacOS)
 * [git](https://git-scm.com/downloads)

1. Install [Geode CLI](https://github.com/geode-sdk/cli/releases/latest) [(Instructions)](/geode/installcli)
    * Unzip the download file somewhere, and [add the directory with geode.exe to your user's PATH variable](/geode/installcli#adding-cli-to-path-on-windows). You may need to restart your command line after adding the directory to your path.

2. [Install Geode to GD](#for-users)

3. Run `geode sdk install` to install the SDK (You can provide an argument to `geode sdk install <path>` if you want to install somewhere other than the default path)
    * The installation should automatically set the `GEODE_SDK` environment variable to point to the SDK on Windows. You might need to restart your terminal or IDE, or even have to restart your computer. If it still doesn't work then you'll have to set it manually, as it is required.

4. Run `geode config setup` from the command line to set up a profile

5. Run `geode sdk install-binaries` to install prebuilt binaries, or [build Geode yourself](/source/building.md)
    * You can pick whether you want the `nightly` or `stable` branch of Geode. `nightly` is the latest commit of the `main` branch, and contains all of the latest and greatest features but may also feature crashes and possibly not even compile, whereas `stable` is the latest released version. Use `geode sdk update nightly` or `geode sdk update stable` to switch between the two branches.

6. All done :) See the [instructions for creating a mod](/geode/creating.md).

