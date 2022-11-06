# Installing Geode (for Users)

> :warning: Please note that Geode is **incompatible with all other mod loaders and mods**. We're porting stuff like MHv7 and BetterEdit over, but don't expect their old releases to work.

## Using the installer

The official Geode installer is currently under heavy remodelling. It will be brought back here when it's ready :) For now, you will have to install the Geode loader manually.

## Manually (on Windows)

1. Download [the latest release](https://github.com/geode-sdk/geode/releases/latest) (download the `geode-vX.X.X-win` zip)

2. Place the ZIP's contents to the root Geometry Dash directory (where `GeometryDash.exe` is)

3. Open the game to make sure it works (if it doesn't, **whitelist the GD folder in your antivirus**)

## Manually (on MacOS)

1. Download [the latest release](https://github.com/geode-sdk/geode/releases/latest) (download the `geode-vX.X.X-mac` zip)

2. Rename `libfmod.dylib` inside `Contents/Frameworks` into `libfmod.dylib.original` if you ever need to uninstall Geode

3. Place the ZIP's contents to the `Contents/Frameworks` directory inside `Geometry Dash.app`

4. Open the game to make sure it works (if it doesn't, **whitelist the GD folder in your antivirus**)



# Installing Geode SDK (for Developers)

## Using the installer

The official Geode installer is currently under heavy remodelling. It will be brought back here when it's ready :) For now, you will have to install the SDK manually.

## Using Geode CLI

### Prerequisites

 * [CMake](https://cmake.org/download/) (minimum v3.13.4, prefer latest)
 * A supported C++ compiler ([clang](https://releases.llvm.org/)/[MSVC](https://visualstudio.microsoft.com/downloads/))
 * [git](https://git-scm.com/downloads)

1. Install [Geode CLI](https://github.com/geode-sdk/cli/releases/latest) [(Instructons)](/docs/info/installcli.md)

  * Unzip the download file somewhere, and [add the directory with geode.exe to your user's PATH variable](/docs/info/installcli.md?id=adding-cli-to-path-on-windows). You may need to restart your command line after adding the directory to your path.

2. [Install Geode to GD](#installing-geode-for-users)

3. Run `geode config setup` from the command line to set up Geode CLI

4. Run `geode sdk install` to install the SDK (You can provide an argument to `geode sdk install <path>` if you want to install somewhere other than the default path)

  * The installation should automatically set the `GEODE_SDK` environment variable to point to the SDK on Windows, but if it doesn't, **you should set it yourself**. You may want to restart your terminal, editors, and possibly computer after setting it.

5. Run `geode sdk install-binaries` to install prebuilt binaries, or [build Geode yourself](/docs/source/building.md)

  * You can pick whether you want the `nightly` or `stable` branch of Geode. `nightly` is the tip of the `main` branch, and contains all of the latest and greatest features but may also feature crashes and possibly not even compile, whereas `stable` is the latest released version. Use `geode sdk update nightly` or `geode sdk update stable` to switch between the two branches.

6. All done :) See the [instructions for creating a mod](/docs/info/creating.md).

