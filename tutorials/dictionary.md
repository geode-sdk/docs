# GD Modding Dictionary

As GD modding is a very specialized field, GD modders use a lot of specialized terminology that can be confusing for a newcomer. This dictionary lists and explains many common (and uncommon) terms used by GD modders, to help new modders better understand this weird world.

## Address

The location in the [raw program binary](#binary) where a function or other program data is found.

## Assembly

A "human-readable" form of [binary code](#binary).

> Further reading: [Wikipedia](https://en.m.wikipedia.org/wiki/Assembly_language)

## Binary

Depending on context, may refer to the **program binary** of GD, aka GD's raw machine code found in `GeometryDash.exe` on Windows, or just **binary data** aka 1s and 0s in general.

> Further reading: [Wikipedia](https://en.m.wikipedia.org/wiki/Machine_code)

## CacaoSDK

An old modding framework that merged with [Lilac](#lilac) and formed [Geode](#geode).

## Calling Convention

A source of pain and suffering on Windows. Dictates whether and how function parameters are passed through [registers](#register) and [the stack](#stack). Common calling conventions used in GD include [thiscall](#thiscall), [optcall](#optcall), and [membercall](#membercall).

## CappuccinoSDK

A library consisting of Cocos2d headers with a few RobTop modifications listed, used even before [traditional modding](#traditional-modding) got its proper start. **A fully outdated and deprecated library.**

> Related words: [Geode](#geode), [Traditional modding](#traditional-modding)

## Cheat Engine

A [debugger](#debugger) commonly used by Windows modders.

> [Homepage](https://cheatengine.org/)

## Cocos2d

The game engine used by GD. The version used by GD is Cocos2d-x v2.2.3, however with many propietary modifications by RobTop, meaning that you need to use a specially prepared version that accounts for these changes such as [Geode](#geode) or [cocos-headers](#cocos-headers) in order to make mods.

> [Homepage](https://github.com/cocos2d/cocos2d-x/tree/cocos2d-x-2.2.3/)

## Cocos-headers

A popular library used in [traditional mods](#traditional-modding) consisting of Cocos2d headers with many of RobTop's modifications added.

> [Homepage](https://github.com/HJfod/cocos-headers)

## Debugger

A tool used for debugging another application (usually GD) for issues (usually crashes), such as [x64dbg](#x64dbg) or [Cheat Engine](#cheat-engine).

> Further reading: [Wikipedia](https://en.m.wikipedia.org/wiki/Debugger)

## Detour

The code you wrote that is run instead of the original function in a [hook](#hooking).

> Related words: [Trampoline](#trampoline), [Hooking](#hooking)

## Director

The central [manager class](#manager) in Cocos2d that manages the current [scene](#scene) and the nodes in it.

> Further reading: [Cocos2d docs](https://docs.cocos2d-x.org/cocos2d-x/v4/en/basic_concepts/director.html)

> Related words: [Node tree](#node-tree), [Scene](#scene), [Node](#node), [Menu](#menu), [Popup](#popup), [Layer](#layer)

## Extension

A different name for mods that is usually used to refer to [Mega Hack](#mega-hack) extensions.

## GD.h

A popular library used in [traditional mods](#traditional-modding) that contains many GD functions and classes. Originally started by [PoweredByPie](https://github.com/poweredbypie), but the most popular version is the fork by [HJfod](https://github.com/HJfod).

> [Homepage](https://github.com/HJfod/gd.h)

> Related words: [Geode](#geode), [Traditional modding](#traditional-modding)

## Geode

A framework meant as a replacement for [traditional modding](#traditional-modding) with the aim of standardizing many aspects of modding and fixing [mod incompatabilities](#mod-incompatabilities).

> [Homepage](https://github.com/geode-sdk/geode)

## Geometry Dash

It's the game.

## Ghidra

A free tool many modders use for [reverse engineering](#reverse-engineering).

> [Homepage](https://ghidra-sre.org/)

## .GMD Files

A file format for storing GD levels, popularized by [GDShare](https://github.com/HJfod/GDShare-mod/). The level data is stored in plain text as its [Plist data](#plist-files) with a surrounding `<d>` tag.

## .GMD2 Files

Another, more complex file format for storing GD levels, popularized by [GDShare](https://github.com/HJfod/GDShare-mod/). Stored as a ZIP file instead of plain text.

> Related words: [.GMD Files](#gmd-files)

## Hooking

Hijacking the execution of a function in GD and redirecting it to [your own function](#detour). The part of a mod that actually does the modifying; the most fundamental modding concept alongside [patching](#patching).

> Further reading: [Handbook](/handbook/vol1/chap1_2.md), [Wikipedia](https://en.m.wikipedia.org/wiki/Hooking)

> Related words: [Detour](#detour), [Patching](#patching), [Trampoline](#trampoline)

## Hook Conflict

A [race condition](#race-condition) where two mods try to hook the same address at the same time, but the hooking library used doesn't provide thread safety. The result is usually one of the hooks mysteriously disappearing. A problem often found in mods that statically link [MinHook](#minhook).

> Related words: [Geode](#geode), [MinHook](#minhook), [Race Condition](#race-condition)

## Hyperdash

An old, never-released framework for creating GD mods. An important influencer that seminally shaped how GD modding looks.

## IDA Pro

A [paid](https://www.youtube.com/watch?v=i8ju_10NkGY) tool many modders use for [reverse engineering](#reverse-engineering).

> [Homepage](https://hex-rays.com/IDA-pro/)

## Layer

A distinct self-contained piece of UI on screen, for example the main menu, the icon select layer, or the move trigger edit popup. Usually inherits from the `CCLayer` class. In any [scene](#scene), there is usually one layer at the bottom of the node tree with potentially [popups](#popup) on top.

> Related words: [Node tree](#node-tree), [Scene](#scene), [Node](#node), [Menu](#menu), [Popup](#popup)

## Lilac

An old modding framework that merged with [Cacao](#cacaosdk) and formed [Geode](#geode).

## Manager

A class that only has a single instance at all times, which is acquired usually through a static getter method in the class itself. Common examples include `GameManager`, which manages everything in GD; `GameLevelManager`, which manages levels; and `CCDirector`, which [manages the node tree](#director).

## Mega Hack

The most popular GD mod of all time, developed by [Absolute](https://absolllute.com/). Also functions as one of the most popular [mod loaders](#mod-loader).

> [Homepage](https://absolllute.com/)

## Membercall

An undocumented calling convention used on Windows, which is a merge between [thiscall](#thiscall) and [optcall](#optcall). Used by the vast majority of functions in GD.

## Menu

An instance of the `CCMenu` class that holds interactive elements such as buttons. Required for buttons that inherit from the `CCMenuItem` class to work, so in practice virtually all buttons in Geometry Dash. This means that every button on screen is the direct child of some menu.

> Related words: [Node tree](#node-tree), [Scene](#scene), [Node](#node), [Layer](#layer), [Popup](#popup)

## MinHook

A popular library used for [hooking](#hooking) in [traditional mods](#traditional-modding).

> [Homepage](https://github.com/TsudaKageyu/minhook)

## Mod

A program that modifies GD by adding, changing, and removing content using various techniques such as [hooking](#hooking) and [patching](#patching). [Traditionally made](#traditional-modding) using libraries such as [gd.h](#gdh) and [cocos-headers](#cocos-headers), but may also be made using a framework such as [Geode](#geode).

## Mod Incompatabilities

When two mods become incompatible with each other, for example through a [hook conflict](#hook-conflict) or [accessing the node tree through absolute indices](/tutorials/nodetree.md).

## Mod Loader

A mod that loads other mods.

## Node

An instance of the `CCNode` class or its subclass. A singular visual element on screen, for example a button or a label. [Has a parent, and may have multiple children](#node-tree). Nodes usually utilize child nodes to form UI components over drawing their own content; for example, a button consists of a child background sprite and a label instead of drawing those in its own logic. A node may have a position, rotation, scale, etc..

> Related words: [Node tree](#node-tree), [Scene](#scene), [Layer](#layer), [Menu](#menu), [Popup](#popup)

## Node Tree

The hierarchy of [nodes](#node) that determines the order of visual elements on screen. At the top of the node tree sits a [scene](#scene), which contains child [layers](#layer) that in turn contain other child nodes.

> Further reading: [Cocos2d docs](https://docs.cocos2d-x.org/cocos2d-x/v4/en/basic_concepts/scene.html)

## Optcall

An undocumented calling convention used on Windows where the first and second parameters are passed through ECX and EDX and the rest through the stack, except for some floating point numbers which are passed through XMM registers. Used by many functions in GD.

> Further reading: [This post on the GD Programming Discord Server](https://discord.com/channels/646101505417674758/881129124335333386/1018079400186621993)

## Patching

Also known as **byte patching**; the act of directly changing GD's program code by overwriting bytes in [the raw binary](#binary). Often used to do things like simple bug fixes and bypassing anticheat. The most fundamental modding concept alongside [hooking](#hooking).

> Related words: [Binary](#binary)

## .Plist Files

A file format used by GD, invented by Apple, that stores data as XML organized into key-value pairs.

> Further reading: [Wikipedia](https://en.m.wikipedia.org/wiki/Property_list)

## Popup

A [layer](#layer) that visually sits on top of another layer. Usually inherits from the `FLAlertLayer` class.

> Related words: [Node tree](#node-tree), [Scene](#scene), [Node](#node), [Layer](#layer), [Menu](#menu)

## Race Condition

When two programs try to affect the same thing simultaniously and the result depends on the order in which they finish. Most commonly encountered in the form of [hook conflicts](#hook-conflict).

> Further reading: [Wikipedia](https://en.m.wikipedia.org/wiki/Race_condition)

## Register

A fixed-length memory location in a CPU that can store a small amount of data. Usually, for a CPU to operate on data that data has to be stored in a register.

> Further reading: [Wikipedia](https://en.m.wikipedia.org/wiki/Processor_register)

## Reverse Engineering

The process of disassembling, decompiling, and understanding how GD works. Usually done using specialized tools such as [Ghidra](#ghidra) or [IDA Pro](#ida-pro), but may also be done using [debuggers](#debugger), trial-and-error modding, or just playing GD.

> Further reading: [Wikipedia](https://en.m.wikipedia.org/wiki/Reverse_engineering)

## Scene

The node that is at the top of the [node tree](#node-tree). There may only be one scene visible at a time, although that scene may technically contain other scenes as children. All visible nodes on screen have the current scene as their ancestor\*. Switching between scenes is done through the [director](#director) and usually involves a [transition](#transition).

\*There is a way in Cocos2d to make visible nodes without adding them to the scene, but this is not used by GD and rarely used in mods.

> Related words: [Node tree](#node-tree), [Popup](#popup), [Node](#node), [Layer](#layer), [Menu](#menu)

## Stack

A last-in first-out datastructure used for storing program memory. Way too complex of a topic for a simple dictionary.

> Further reading: [Wikipedia](https://en.m.wikipedia.org/wiki/Stack_(abstract_data_type)), [Stack Overflow](https://stackoverflow.com/questions/556714/how-does-the-stack-work-in-assembly-language)

## Thiscall

A calling convention used on Windows where the first parameter is passed through ECX and the rest through the stack. Used by most functions in Cocos2d.

## Traditional Modding

Mods made and modding done without a dedicated framework like [Geode](#geode), usually using a combination of [gd.h](#gdh), [cocos-headers](#cocos-headers), and [MinHook](#minhook).

## Trampoline

Also known as "the original function", it is what you call to invoke the behaviour of the function you overwrote in a [hook](#hooking).

> Related words: [Detour](#detour), [Hooking](#hooking)

## Transition

An animation that is played when switching from one [scene](#scene) to another. In GD, this is nearly always a fade transition. Inherits from the `CCTransition` class.

Also, what most GD modders do once they become experts.

> Related words: [Node tree](#node-tree), [Scene](#scene), [Node](#node), [Menu](#menu), [Popup](#popup), [Layer](#layer)

## Undefined behaviour (UB)

Code where the C++ standard does not say what should happen, and as such the results are completely unpredictable. Your code is most likely full of this.

> Further reading: [cppreference.com](https://en.cppreference.com/w/cpp/language/ub)

## x64dbg

A [debugger](#debugger) commonly used by Windows modders.

> [Homepage](https://x64dbg.com/)
