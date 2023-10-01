---
title: Getting started (old)
icon: play
description: Instructions for creating mods with the Geode SDK & other developer tools
order: 1
---

# Getting started

Make sure to [install Geode](/installation.md) first.

## Creating a new mod

Navigate to some projects folder on your computer and run `geode new` from the terminal. Follow the prompts given to initialize a new directory with template mod code, and then open up the created directory in your favorite code editor.

The structure of a Geode mod is the following:

 * `mod.json` contains metadata about your mod, such as the name, version, settings, etc. [See detailed info](/mods/configuring.md)
 * `about.md` is a free-form description about the mod, like a Github README. [See detailed info](/mods/md-files.md)
 * `CMakeLists.txt` is for configuring CMake. The default configuration provided by `geode new` should cover most smaller mods
 * `src` is the directory where your mod's source code lives

In addition to these predefined files, you may also find common additional ones:
 
 * `changelog.md` lists all of the changes between versions to the mod; [see detailed info](/mods/md-files.md#changelogmd)
 * `support.md` is free-form info about how to show support to the developer of the mod; [see detailed info](/mods/md-files.md#supportmd)
 * `resources` is a common name for the directory containing any assets like sprites and sounds for the mod
 * `include` is a common name for the directory containing header files for [mods with an API](/mods/dependencies.md)

## Writing code

> :information_source: If you haven't written GD mods before, see [the Handbook](/handbook/chap0.md) for a tutorial

Modding in Geode is for the most part the same as traditional modding, however there are a few things where it differs. Here are a list of Geode-specific concepts you might like to familiarize yourself with:

 * [Hooking with `$modify`](/tutorials/modify.md) and [adding fields](/tutorials/fields.md)
 * [Using sprites and other resources](/mods/resources.md)
 * [Settings](/mods/settings.md) and [saving data](/mods/savedata.md)
 * [String IDs](/tutorials/nodetree.md) and [Layouts](/tutorials/layouts.md)
 * [Events](/tutorials/events.md)
 * [Using dependencies](/mods/dependencies.md)
 * [Publishing mods](/mods/publishing.md)

See also [our tutorial](/tutorials/migrating.md) for porting traditional mods to Geode.

The [Tutorials](/tutorials) category in general is a great source for information about working with Geode mods.

## Building mods

You can build mods using CMake. If you're using VS Code with the [CMake Tools extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools), simply run `CMake: Configure` from the VS Code command palette (press F1 to launch it) to configure the mod. **On Windows, make sure to select an x86 generator**, for example `Visual Studio Community 2019 Release - amd64_x86`. Note that for Visual Studio generators, the second number is the target architecture, which must be x86, so `x86_amd64` is the _wrong_ one.

After configuring, if you're **on Windows**, run `CMake: Select Variant` (select either `Release` or `RelWithDebInfo`). This may also be done through the bottom status bar. There should be a button that displays `Debug`. Click on this and change it to the option in the dropdown that displays `Release`.

Now you can build the mod by running `CMake: Build` or pressing the `Build` button in the bottom status bar. If the building is succesful, the mod should be automatically installed in GD. Open up GD and see if the mod works.

If you don't want to use VS Code, you can also use CMake from the command line with `cmake -B build -T host=x64 -A win32` and `cmake --build build --config Release` **(on Windows)**.
