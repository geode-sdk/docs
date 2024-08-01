# Chapter 2.5: Finding Callbacks

The reason we started reverse engineering by finding `init` functions is because they're the most common thing you'll be hooking. However, there is another type of function that is used very often in mods - callbacks!

Callbacks are passed to buttons through their `create` function (usually `CCMenuItemSpriteExtra::create`), and are what the button does when it is clicked - vitally important for any mod which wants to change what a button does!

Callbacks are usually named `onSomething` and take a single `CCObject*` parameter along with the current class. This is what passing a callback looks like in C++ code:

```cpp
auto button = CCMenuItemSpriteExtra::create(
    ..., this, menu_selector(ThisClass::onButtonClicked)
);
```

The `menu_selector` macro is just syntactic sugar for `reinterpret_cast<cocos2d::SEL_MenuHandler>(&ThisClass::onButtonClicked)` - in other words, it just casts the callback to the correct type.

What we can take away from this code is that to find a callback in Ghidra, all we need to do is find out where the button it's used on is created, and then look at that button's create call. The easiest way to find out where our desired callback is used is through Android - although, first we need to figure out which callback we're looking for.

Let's say we're looking for whatever function it is that shows this popup: level stats.

![Level statistics popup for Zenith by HJfod](/assets/handbook/vol2/LevelStats.png)

As a start, thanks to DevTools, we know this layer's name is `LevelInfoLayer` (todo: add picture), so we're looking for some callback in it.

Now, we know that the button that shows this has an info button sprite, whose sprite name is `GJ_infoIcon_001.png` (thanks, Geode VS Code extension).

![The info sprite found through the Geode VS Code extension, showing that its name is `GJ_infoIcon_001.png`](/assets/handbook/vol2/infospritename.png)

Let's find `LevelInfoLayer::init` and then find where it uses the info sprite:

![Code showing a `CCSprite` being created with `GJ_infoIcon_001.png`](/assets/handbook/vol2/infoiconuse.png)

Hm, this doesn't seem right. Let's check the unknown function call after:

![Code resembling a `create` function](/assets/handbook/vol2/CCMenuItemSpriteExtra_create.png)

Hm, this looks like a `create` function. Checking the constructor, it seems to be `CCMenuItemSpriteExtra::create` - let's rename it as such. Checking on Android, the function has 4 parameters: `CCNode*`, `CCNode*`, `CCObject*`, and `void(CCObject::*)(CCObject*)`.

The first three are easy - however, the last one is a function pointer. Cocos2d has an alias for this pointer in particular - `SEL_MenuHandler`. However, as it's a typedef, it doesn't come with any symbols and as such we need to add it manually:

![Adding a new function definition in Ghidra](/assets/handbook/vol2/ghidra_typedef.png)

![Defining SEL_MenuHandler in Ghidra](/assets/handbook/vol2/SEL_MenuHandler_def.png)

After this, we can correct the signature of `CCMenuItemSpriteExtra::create`.

![The correct signature for `CCMenuItemSpriteExtra::create` in Ghidra](/assets/handbook/vol2/CCMenuItemSpriteExtra_create_correct.png)

> :warning: Note the calling convention being `__fastcall` rather than `__thiscall` - this will be explained later!

Now if we look back at `LevelInfoLayer::init`, we can see that the function call looks more accurate - there's a function being passed as a parameter now! If we follow through to that function's definition, we will see that it in fact is the one we were looking for, judging by the string literals present.

![A call to `CCMenuItemSpriteExtra::create` in Ghidra with the correct args](/assets/handbook/vol2/fixed_button_params.png)

![`LevelInfoLayer::onLevelInfo`](/assets/handbook/vol2/callback.png)

Now we just need to find the same function on Android to get the correct signature. Here we can take advantage of the fact that Ghidra can search for string literals defined in the code:

![Searching for the string `Total Attempts` in Ghidra](/assets/handbook/vol2/ghidra_string_search.png)

Filtering through the potential XREFs, we find `LevelInfoLayer::onLevelInfo`, which checking through `LevelInfoLayer::init` is the correct one. So we can retype our Windows function and be confident we found the correct one!

Right now, reverse engineering should seem a little less mystifying, however there is one more important aspect we need to consider: calling conventions. Where do they come from? Where do they go? How do you know what calling convention a function uses?

Before we can delve into that, we first need to learn a bit about assembly:

[Chapter 2.6: Introduction to Assembly](/handbook/vol2/chap2_6.md)
