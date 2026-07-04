# Chapter 2.1: Reverse Engineering

At this point in this handbook, **reverse engineering** (RE for short) has been alluded to and referred to numerous times. At its core, it is quite simply **the process of figuring out how something works**. However, in practice with GD, this often involves having to read assembly code, understanding how your computer works, and a lot of other deeply low-level stuff. And even so, it is fundamental to making mods, as **you can't modify something you don't understand**. This is why this handbook has an entire dedicated Volume just for reverse engineering; **it is difficult, it is complicated, and yet it's super important**. Every modder has to learn reverse engineering at some point or another, and it's best to start early.

It should be noted that reverse engineering is also an ever-evolving skill; you're **never going to be done learning it**. This Volume covers many important aspects of it, but it should by no means be treated as a comprehensive list.

But that's enough preface. Let's start actually **REing**!

## Tools

There are a lot of tools GD modders use for reverse engineering, but some of the most common ones include:

 * [Ghidra](https://github.com/NationalSecurityAgency/ghidra)
 * [IDA Pro](https://hex-rays.com/ida-pro) (which every modder most definitely has legally bought :wink:)
 * [x64dbg](https://x64dbg.com/) (Windows)
 * [Cheat Engine](https://cheatengine.org/) (Windows)
 * [ReClass](https://github.com/ReClassNET/ReClass.NET) (Windows)
 * [DevTools](https://github.com/geode-sdk/DevTools) (Traditionally [CocosExplorer](https://github.com/matcool/CocosExplorer))
 * [Slicer](https://github.com/zorgiepoo/Bit-Slicer) (Mac)
 * [LLDB](https://lldb.llvm.org/)

For this tutorial, we will be using **Ghidra** and **x64dbg**.

## Setting Up Ghidra

First, [download Ghidra](https://github.com/NationalSecurityAgency/ghidra/releases/latest) and install it on your machine. Open it, **create a new project**, and you should see something like this:

![Image showing the project page of Ghidra](/assets/handbook/vol2/ghidra_start.png)

> :information_source: You can switch themes by going in `Edit > Theme > Switch...`

Now, drag Geometry Dash's binary (e.g. `GeometryDash.exe`) into the window, add it with the default settings, open it, and **Analyze it** with the default settings (if you're reverse engineering Android, disable the `Non returning functions: discovered` option). Depending on your computer, analyzing might take a while - go grab another cup of orange juice while waiting for it to finish.

You should now see a window like this:

![Image showing the opened project in Ghidra](/assets/handbook/vol2/ghidra_window.png)

For macOS Geometry Dash, Ghidra will ask how it should import the application. You will typically want to import both the x64 and arm64 versions of macOS, so "Batch Import" is the fastest way to get there.

The three most important windows for our purposes are **Symbol Tree**, **Listing**, and **Decompiler**. You can close the others.

## Scripts Intermission

Ghidra comes with the ability to run scripts, which may be useful while reverse engineering. You can access the Script Manager through the menu bar `Window > Script Manager` or through its icon on the main toolbar:

![Image showing the Script Manager icon, which is a green circle with a white triangle inside](/assets/handbook/vol2/ghidra_scriptmanager_icon.png)

If you have checked out the [Geode bindings](https://github.com/geode-sdk/bindings/) to a local directory (which you should if you're planning on adding functions to them), you will also want to tell Ghidra to look at its scripts directory. To do so, using the Script Manager window, click the "Manage Script Directories" button in the toolbar. Then, add the path to the binding's scripts directory (it should end with `/bindings/scripts/ghidra`). Click the Refresh button at the top of that window to finish your changes.

If you have done this correctly, you should see a "GeodeSDK" folder in the Script Manager:

![Screenshot of the Script Manager after bindings scripts have been added](/assets/handbook/vol2/ghidra_scriptmanager_post_setup.png)

### RecoverClassesFromRTTI (Windows)

This script recovers some class information, including basic layout and virtual functions, through information in the binary. Depending on your hardware, this script may take a while to complete. 

![Image showing the Symbol Tree after the script is executed](/assets/handbook/vol2/ghidra_symboltree_post_rtti.png)

### FindVtables (macOS)

This script recovers the names of virtual function tables through information in the binary.

![Image showing the Symbol Tree after the script is executed](/assets/handbook/vol2/ghidra_symboltree_post_vtables.png)

### SyncBroma

This script matches addresses in the Geode bindings to addresses within the binary and updates class layouts. For now, you can leave all of the options at their defaults. Depending on how adventurous you are feeling, you may want to skip executing this script for now, as it makes the rest of the handbook useless...

> :information_source: None of these scripts are perfect and may produce incorrect output. Do not trust the Decompiler window to be a source of truth.

Now that we have Ghidra setup, it's time to learn how we can find a layer's `init` function.

[Chapter 2.2: Finding `MenuLayer::init`](/handbook/vol2/chap2_2)
