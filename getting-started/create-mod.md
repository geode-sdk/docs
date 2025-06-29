---
title: 3. Creating and building a new mod
order: 5
---

# Creating a new mod

After all that setup, you can now create your first mod!

To do this, open up a terminal where you want to create your project and run:
```bash
geode new
```
Follow the given prompts and afterwards you should have a new folder containing the code for your mod.

## Files

You may notice the project already comes with a few files. Lets go over them:
 * `CMakeLists.txt` - This is the main file for your CMake project.
 * `about.md` - Here you can write a very long description page for your mod, in markdown. Think of it as a README for your mod! This file is technically optional, but highly recommended.
 * `logo.png` - This is the icon for your mod, which shows up in-game. This file is technically optional, but highly recommended.
 * `mod.json` - This json file contains all the metadata about your mod, such as name, version, custom resources, settings, etc. [See this page for detailed info](/mods/configuring)

If you plan on releasing your mod, remember to edit the about.md and logo.png files!

The source code for your mod can be found inside the `src` folder.

## Additional Files

Geode will also look for these special files within your mod folder:
 * `changelog.md` - Lists all of the changes between versions to the mod; [see detailed info](/mods/md-files)
 * `support.md` - Free-form info about how to show support to the developer of the mod; [see detailed info](/mods/md-files)

# Build

Now, to build your mod you have a few options:

If youre using an IDE such as Clion, VScode or Visual Studio, head over to the [IDE Setup](/getting-started/ide-setup) page.

If you're building for Android, check out the [Android section](#build-for-android).

Otherwise if you want to build your mods manually from the command line you can do that by simply running these commands in your mod's folder:
```bash
# Configures & builds for the current platform
geode build
```

> Check out `geode build --help` for other options!

If you have an issue running that command for whatever reason ([do let us know!](https://github.com/geode-sdk/cli/issues)), you can build your mod the same way using these commands:
```bash
# Configure CMake
cmake -B build

# Build the project
cmake --build build --config RelWithDebInfo
```

If you [created a profile in CLI](/getting-started/geode-cli), then the mod should be automatically installed to GD. If not, then the built `your.mod.geode` package should be in your build folder, from where you can manually install it in-game.

## Build for Android

To build mods for Android you must install the [Android NDK](https://developer.android.com/ndk/downloads). Extract it somewhere and set the `ANDROID_NDK_ROOT` enviroment variable to its path.

> On **Windows** you must also install [Ninja](https://github.com/ninja-build/ninja/releases). If you have Scoop, you can do this via `scoop install ninja`.

You must also get the Geode binaries for Android, you can do this using the CLI:
```bash
geode sdk install-binaries -p android64
# Or if you're using a 32 bit phone
geode sdk install-binaries -p android32
```

Now you can build your mod for Android via:
```bash
geode build -p android64
# Or if you're using a 32 bit phone
geode build -p android32
```

You can then copy the built .geode file from the `build-android64` folder to your phone at this location:
```
/storage/emulated/0/Android/media/com.geode.launcher/game/geode/mods/
```

## Building Windows mods on Linux

If you have followed the steps earlier and installed all the required tools with `geode sdk install-linux`, building should be as simple as on Windows:

```bash
geode build
```

If you have installed the Windows SDK and the toolchain in a different way, you will have to provide paths to them manually:

```bash
geode build -- -DCMAKE_TOOLCHAIN_FILE=/path/to/clang-msvc-sdk/clang-msvc.cmake -DSPLAT_DIR=/path/to/splat
```
