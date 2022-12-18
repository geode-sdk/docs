# What is Geode?

## In short, it is to GD what Forge / Fabric are to Minecraft.

Geode is a **modding SDK** and **mod loader**. It contains a suite of tools that make developing mods fast and easy, and also has the actual loader for using those mods.

## What does this mean for players?

### Geode makes installing mods simple

No more need to drag weird .DLL files to barely documented folders, or figuring out what a `settings.txt` file is. **Installing a mod in Geode is as simple as clicking an "Install" button in-game**. Even installing the loader itself is done through a simple [GUI installer](https://github.com/geode-sdk/installer) that manages the installation process for you.

### Geode replaces other mod loaders

Geode fully manages all the mods it loads, and as such is **incompatible with all other mod loaders and mods**. You can't use something like **QuickLdr** or **MHv7 extensions** (Mega Hack itself will be ported over though) alongside Geode.

### What about non-Geode mods?

Mods made using traditional methods, such as GD Hacker Mode or BetterEdit v4 are unfortunately **never going to be supported** by Geode. Geode does not, and frankly, **can not support other mod loaders**. This is because doing so would be fully at odds with what Geode aims to do; if you let mods create hooks using their own methods, traverse the node tree as they wish and handle saving settings and data manually, **you will end up with the same incompatability issues you started with**. Geode will never be able to load arbitary .DLLs, because doing so simply does not make sense.

Of course, this would be quite unhelpful for end users, so **we're doing our best to get as many mods ported to Geode as possible**. BetterEdit, MegaHack v7, ReplayBot, TextureLdr, Run Info and many others **will be ported as Geode mods**, and ones that aren't will be receiving **alternatives** (for example, an alternative to Sai Mod Pack is planned very soon). If there's a mod you'd like to see ported to Geode, [let us know](https://discord.gg/9e43WMKzhp).

## What this means for modders

### Geode replaces MinHook and gd.h

Geode acts as replacement for the popular [MinHook](https://github.com/TsudaKageyu/minhook), [gd.h](https://github.com/hjfod/gd.h) and [cocos-headers](https://github.com/HJfod/cocos-headers) toolkit that forms the base for **traditional modding**. Geode comes with its own hooking system, its own Cocos2d headers and its own GD bindings. The main goals of Geode is to **make modding simpler** and to **fix mod incompatability**. For users, Geode replaces traditional mod loaders like Mega Hack v7's extensions folder or QuickLdr.

### Geode is a mod loader

Geode loads mods packaged as `.geode` files. They are actually just ZIPs that have the platform-specific DLLs, but all Geode mods also contain one extra file: `mod.json`, which contains information about the mod.

### Geode makes modding simpler

Want to create a hook? [Geode's `$modify` syntax makes this incredibly simple and easy to read](/tutorials/modify.md).

Want to add custom sprites? Add your resources under the `resources` key in mod.json; All the `UHD` and `HD` variants are automatically created and the resources packaged into your `.geode` file, so you don't have to worry about distributing resources with the DLL anymore.

Want to add settings to your mod? Geode comes with a built-in and very simple solution; just add settings under the `settings` key in your mod's `mod.json` and you're good to go. Using settings is as simple as `Mod::get()->getSettingValue<type>("setting-id-here")`

Want to make async web requests, create list nodes, popups, and much more? Geode is filled with utilities meant to make your code much more concise.

### Geode lets you publish mods

Geode also comes with a simple **publishing platform**; you can publish your mods on the [Geode Mods Index](https://github.com/geode-sdk/mods/), after which **users can install and update mods in-game**. No need to set up a Discord server with a downloads channel containing weird .DLLs and trying to ensure users stay up-to-date; all you have to do is make sure the latest version of your mod is on the Index, and Geode will handle the rest.

### But the main reason for using Geode is mod interoperability.

In traditional modding, what do you do if two mods modify the same layer? Sometimes this happens to work as expected, but more often than not faulty logic in either or both mods leads to **bugs** and **crashes**. And fixing these issues can be more cumbersome than expected. How are you supposed to make sure your mod doesn't glitch out, when you don't even have a reliable way of [traversing the node tree](/tutorials/nodetree.md)? If another mod becomes incompatible, how do you even check for that? You would have to write your own utilities for checking if another `.DLL` is loaded and that can go south very easily.

To solve these problems, the most major thing Geode introduces is **IDs**. Every mod in Geode has an ID, like `geode.loader` or `hjfod.betteredit`. Checking if another mod is loaded is as simple as querying the loader: `Loader::get()->isModLoaded("mod.id")`. If your mod becomes incompatible with another, Geode comes with the tooling to fix that with as little code as possible.

On top of this, Geode also addresses a lot of the sources of incompatability like nodes being added to the wrong places. [Geode comes with string IDs and (soon) automatic layouting tools](/tutorials/nodetree.md), which make writing code that is hard to break very simple. One of the leading philosophies of Geode is that a mod that simply modifies the look of an UI and another that simply adds some buttons into existing menus should **always be compatible**.

### And, well, it's free.

Geode is 100% free and always will be on all platforms. **Our goal is to just make GD modding better** :)
