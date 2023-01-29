---
icon: help-circle
description: An explanation of what Geode is
---

# What is Geode?

![A screenshot of a bug, where the mods BetterEdit and GDShare are conflicting, with a button added by GDShare being misplaced as a result](/assets/BetterEdit_GDShare_bug.png)

> This is an image of two popular mods, GDShare and BetterEdit, infamously conflicting with each other, with the result being misplaced buttons and a lot of disgruntled developers. A huge percentage, if not a majority of all bugs reported in GD mods are caused by **hook conflicts**, **direct node tree access**, and other **mod incompatabilities**. **This is what Geode has been made to solve.**

 * Geode's main goal is to **make using mods less of a pain**; installing mods is simple, and things just work out-of-the-box with no need to mess around with load order.

 * Geode is also **a complete framework**. Instead of having to add `gd.h`, `cocos-headers`, and `MinHook` to every project every single time, Geode has all of the GD and Cocos2d headers in one package, along with a bunch of utilities that make modding simpler.

 * Geode does not impose any limits on what mods you can make; all of the basics, like hooking, creating UI, and the meat of how the mod works are still all the same. **All Geode does is reduce the amount of boilerplate you have to write**, and provide a lot of utilities to ensure your mod works together with other mods.

 * Geode does not load traditional `.DLL` mods; instead, it loads mods packaged as `.geode` files. This does mean that **by default, Geode will not load any old mods**. However, we do plan on making a mod that lets you load old DLLs (and you can also technically install Geode alongside another mod loader, however this will likely break!); with the caveat **that older mods are very likely going to break in many ways**. This is an unfortunate result of the fact that **Geode can only ensure mods built for it work**. Older mods by nature can not be made compatible with each other.

 * We're working on porting as many older mods over to Geode as we can, as that is the only way we can ensure stability. Luckily, this also provides us a golden opportunity to fix long-standing bugs and improve the mods - **BetterEdit**, **TextureLdr**, **GDShare**, **Run Info**, and more will have new, much better versions in Geode. If there's a mod you'd like to see ported to Geode, [do let us know](https://discord.gg/9e43WMKzhp).

 * **In short, it is to GD what Forge / Fabric are to Minecraft.**
