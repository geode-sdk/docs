# Chapter 2.1: Reverse Engineering

At this point in this handbook, **reverse engineering** (RE for short) has been alluded to and referred to numerous times. At its core, it is quite simply **the process of figuring out how something works**. However, in practice with GD, this often involves having to read assembly code, understanding how your computer works, and a lot of other deeply low-level stuff. And even so, it is fundamental to making mods, as **you can't modify something you don't understand**. This is why this handbook has an entire dedicated Volume just for reverse engineering; **it is difficult, it is complicated, and yet it's super important**. Every modder has to learn reverse engineering at some point or another, and it's best to start early.

It should be noted that reverse engineering is also an ever-evolving skill; you're **never going to be done learning it**. This Volume covers many important aspects of it, but it should by no means be treated as a comprehensive list.

But that's enough preface. Let's start actually **REing**!

## Tools

There are a lot of tools GD modders use for reverse engineering, but some of the most common ones include:

 * [Ghidra](https://ghidra-sre.org/)
 * [IDA Pro](https://hex-rays.com/IDA-pro/) (which every modder most definitely has legally bought :wink:)
 * [x32dbg](https://x64dbg.com/) (Windows)
 * [Cheat Engine](https://cheatengine.org/) (Windows)
 * [ReClass](https://github.com/ReClassNET/ReClass.NET) (Windows)
 * [DevTools](https://github.com/geode-sdk/DevTools) (Traditionally [CocosExplorer](https://github.com/matcool/CocosExplorer))
 * [Slicer](https://github.com/zorgiepoo/Bit-Slicer) (Mac)
 * [LLDB](https://lldb.llvm.org/)

For this tutorial, we will be using **Ghidra** and **x32dbg**. 

## Setting Up Ghidra

First, [download Ghidra](https://ghidra-sre.org/) and install it on your machine. Open it, **create a new project**, and you should see something like this: 

![Image showing the project page of Ghidra](/assets/handbook/vol2/ghidra_start.png)

> :information_source: The dark theme used is [Mat Dark](https://cdn.discordapp.com/attachments/651480005536383009/1186912004775092345/Mat_Dark_Theme.theme)

Now, drag Geometry Dash's binary (e.g. `GeometryDash.exe`) into the window, add it with the default settings, open it, and **Analyze it** with the default settings (if you're reverse engineering Android, disable the `Non returning functions: discovered` option). Depending on your computer, analyzing might take a while - go grab another cup of orange juice while waiting for it to finish.

You should now see a window like this:

![Image showing the opened project in Ghidra](/assets/handbook/vol2/ghidra_window.png)

The three most important windows for our purposes are **Symbol Tree**, **Listing**, and **Decompiler**. You can close the others.

Now that we have Ghidra setup, it's time to learn how we can find a layer's `init` function.

[Chapter 2.2: Finding `MenuLayer::init`](/handbook/vol2/chap2_3.md)
