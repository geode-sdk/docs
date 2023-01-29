---
title: How to Use Geode
icon: play
---

# Top 10 Geode Concepts

This is a quick overview of the **most major concepts in Geode**. Some of these are familiar from other modding toolkits, but some are Geode-specific!

## 1. Mods

The central concept in Geode is - quite obviously - the mods. In Geode, these come in the form of `.geode` files - ZIP files that contain all of the mods assets, including platform-specific binaries, resources, and info about the mod, with the most important one being **mod.json**. It is a file that has all the basic information about the mod - its name, developer, a description, and other things you might expect. The most important thing in this file is the **mod ID** - a short human-readable piece of text that uniquely identifies this mod. This ID is used by other mods to check if this mod is loaded, by the loader to see if this mod has updates available, and in every situation where a reference to this mod is needed.

At runtime, every mod also has associated with it an instance of the [Mod](/classes/geode/Mod) class - which the current mod can access by doing `Mod::get()`, and through which everything from placing hooks to patching addresses to saving settings is done.

## 2. Loader

Of course, there has to be something to load the mods, and for that there is the **Geode loader** - an instance of the [Loader](/classes/geode/Loader) class. The loader is the central manager in Geode - it manages all mods, and deals with setting up everything, like crashlogs, resources, etc..

## 3. Codegen & Headers

As Geode's goal is to be cross-platform, its set of GD headers has to be carefully crafted to include definitions for all platforms. For this reason, Geode uses codegen to generate its headers. Another benefit of this is that if Geode is missing a function you need, **adding new bindings is really easy** - you just have to add it to [your local copy of Geode's GeometryDash.bro](https://github.com/geode-sdk/geode/blob/main/bindings/GeometryDash.bro) - the file Geode uses to codegen all of its GD bindings.

## 4. CLI

To help work with Geode mods, Geode comes with [its own command-line tool called Geode CLI](https://github.com/geode-sdk/cli). The CLI handles things like installing & updating the Geode SDK and packaging `.geode` files - and to that end has a lot of really useful functionality. For example, if you want to include resources or custom fonts in your mod, all you have to do is specify them in your **mod.json** and Geode CLI will automatically package all of them for you - no need to mess around with TexturePacker or related tools!

## 5. Hooks

In order for mods to do something, they need to place hooks. If you are new to modding and don't know what that is, [check out the handbook](/handbook/chap0.md). For experienced modders, the way you place hooks in Geode is
```cpp
// this magical syntax is basically just the entry point, like int main()
$on_mod(Loaded) {
    Mod::get()->addHook(Hook::create(
        Mod::get(),
        reinterpret_cast<void*>(geode::base::get() + address),
        &myAwesomeHook,
        "My awesome hook!",
        {},
        {}
    ));
}
```
However, as this is quite verbose and having to manually specify all of your hooks kinda blows, Geode comes with incredibly magical syntactic sugar known as **`$modify`**:
```cpp
class $modify(TargetLayer) {
    bool functionToHook(...) {
        // TargetLayer::functionToHook is now hooked! no other code necessary
    }
};
```

## 6. Events

In order for mods to communicate with each other, the traditional way to do it is by linking to the other mod. While this is still possible in Geode, what if you would like to have an optional dependency - as in, only do stuff related to another mod if it is loaded? For this, the main mechanism Geode introduces are called **events**. Events are basically what their name suggests - you can post events to other mods, and listen for events. Events include things like **mods being loaded**, **settings being edited**, etc..

These are somewhat reminiscent of **delegates** in Cocos2d and GD - however, as events are based on string IDs, can be applied to any existing class, can have any number of listeners, and can transmit any data, they are much more powerful.

## 7. Node IDs

Every mod has to interact with GD's UI in some way, and this will eventually result in the modder asking themselves the eternal question - how do I add my button next to this other button in the UI? Traditionally, the only way to do this has been to traverse the node tree using the absolute indices of child nodes - that is, getting the 3rd child of this menu, then the 5th child of that, then so on and so forth. **This breaks incredibly easily**, as the moment there is any disruption to the amount of children in a node, mods get thrown off.

As this is really common in mods and also one of the most common sources of incompatabilities, Geode provides a much better alternative - **node IDs**. With IDs, instead of getting the 3rd child of a layer, you can just do `this->getChildByID("the-button-im-looking-for")`. This is much more consistent, as you can be pretty much certain that this will never fail - and you don't have to write any sort of property-based node lookup code to verify that is the case!

On top of this, node IDs also enable other mods to safely rewamp scenes without fear of breaking other mods - if you want to make a mod that changes the layout of [MenuLayer](/classes/MenuLayer), all you have to make sure is that all of the node IDs are still there, and you can rest assured that other mods will work - assuming they aren't using absolute child indices for some reason.



## 8. Index

Geode comes with an in-game list, from where users can download & update mods at the click of a button. Once your awesome mod is ready for publishing, you should [submit it to the Geode index so it appears on the list](https://github.com/geode-sdk/mods). This way, you don't have to worry about setting up servers to distribute your mods and explain how to install your mod to users - we will handle that for you :)

## 9. Developer Tools

Geode's development aid is not limited to just the framework and CLI - Geode also has [its own VS Code extension](https://marketplace.visualstudio.com/items?itemName=GeodeSDK.geode) that comes with a bunch of useful tools for working with GD mods, and [a developer tools mod](https://github.com/geode-sdk/DevTools) for inspecting layers in-game.

## 10. Docs

One final minor goal of Geode is to **spread modding knowledge more**. GD modding has long been plagued by a lack of tutorials, documentation, and examples - and we want to change that. The goal of Geode docs is not only to explain how Geode works, but to act as a learning resource for GD modding in general, so that future modders won't have to learn everything by themselves through trial-and-error like we did. To that end, we have automated documentation for all GD and Cocos2d classes here on the docs site, tutorials for common things like [making popups](/tutorials/popup.md) and [creating buttons](/tutorials/buttons.md), and [a handbook that teaches everything there is to know about GD modding from the ground up](/handbook/chap0.md).

If you are interested in trying Geode out for yourself, see [Installation](/installation.md) for installation instructions, and [Quick Start](/quickstart.md) for getting started with developing mods for Geode.
