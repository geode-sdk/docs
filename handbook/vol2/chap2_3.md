# Chapter 2.3: Finding `LevelSettingsLayer::init`

Not all `init` functions are found on the vtable however. The only reason `MenuLayer::init` is on there is because it takes no extra parameters - so it overrides `CCNode::init`. However, if the init function has any parameters, it has a different name, or the class doesn't inherit from `CCNode`, then you most likely won't find it on the vtable (unless it is explicitly marked virtual).

Luckily, finding non-virtual `init` functions is still quite easy! As an example, let's find the `init` function for `LevelSettingsLayer`!

![Picture showing LevelSettingsLayer open in the Ghidra class list](/assets/handbook/vol2/LevelSettingsLayer_in_tree.png)

Let's start as before by finding `LevelSettingsLayer` in the Ghidra class list (tip: you can use the search function at the bottom!). Click any of the vtables to navigate to that vtable in the disassembly view.

![Picture showing a vtable for LevelSettingsLayer, with two XREFs](/assets/handbook/vol2/LevelSettingsLayer_vtable_ctor_xref.png)

Now, check the XREFs - for most classes, there should be two [[Note 1]](#notes). One of these is the constructor, and the other is the destructor. Pick one, and see which one it is - we are looking for the constructor.

![Picture showing one of the XREFs for LevelSettingsLayer open in the decompiler, likely being the constructor](/assets/handbook/vol2/LevelSettingsLayer_ctor.png)

Ah, this is likely the constructor! You can tell this by the fact that it initializes a bunch of fields in the class pointer to their default values, as well as that it contains a single call to the base constructor. Although, the base constructor is unknown - double-click it to find out what it is!

![Picture showing FLAlertLayer's constructor open in the decompiler](/assets/handbook/vol2/FLAlertLayer_ctor.png)

Judging by the vtables, this seems to quite clearly be `FLAlertLayer`'s constructor. So let's rename the function to `FLAlertLayer::FLAlertLayer` and press `Alt + Left Arrow` to go back to the previous constructor (note: you may have to press it multiple times because Ghidra is quirky like that).

![Picture showing LevelSettingsLayer constructor renamed to the correct name](/assets/handbook/vol2/LevelSettingsLayer_ctor_renamed.png)

Now check the XREFs for the `LevelSettingsLayer` constructor. There's likely only one [[Note 1]](#notes) - click it to follow the reference.

![Picture showing `LevelSettingsLayer::create`](/assets/handbook/vol2/LevelSettingsLayer_create.png)

Hm, okay, let's analyze this code a little bit. First it calls `operator new`, then it checks if that was null, then calls a function on the returned value, and if that function returned a truthy value, it calls `autorelease` and returns the value.

This looks very familiar - this is the structure of a `create` function:

```cpp
LevelSettingsLayer* LevelSettingsLayer::create(...) {
    auto ret = new LevelSettingsLayer();
    if (ret) {
        if (ret->init(...)) {
            ret->autorelease();
            return ret;
        }
        delete ret;
    }
    return nullptr;
}
```

We can be almost certain that this is the `create` function, which means that the one unknown function call we have before the call to `autorelease` must be `LevelSettingsLayer::init` - double click the function to make sure!

![Picture showing an unknown function call at the start of what we assume to be `LevelSettingsLayer::init`](/assets/handbook/vol2/LevelSettingsLayer_base_init_call.png)

Hm, there's a call to some unknown function at the start, whose return value the code checks to be truthy before doing anything else, and if it's false then it does nothing but propagate that information up to the original caller. Sounds very similar to the structure of an `init` function:

```cpp
bool LevelSettingsLayer::init(...) {
    if (!FLAlertLayer::init(...))
        return false;

    ...

    return true;
}
```

Let's make sure by checking out the unknown function's body:

![Picture showing what is clearly `FLAlertLayer::init`](/assets/handbook/vol2/FLAlertLayer_init.png)

OK, this is clearly an `init` function. Since we know from the constructor that the base class of `LevelSettingsLayer` is `FLAlertLayer`, that means that this is most likely `FLAlertLayer::init` [[Note 2]](#notes). That means that the other function we found is very likely `LevelSettingsLayer::init` as well, so let's rename both approppriately:

![Picture showing fixing `LevelSettingsLayer::init`'s signature](/assets/handbook/vol2/LevelSettingsLayer_init_rename.png)

Well. Hmm. We know that any `init` function returns a bool, but what about these parameters? `LevelSettingsLayer` probably doesn't actually take a bare `CCObject*` and `undefined4` is definitely not the correct type - but how do we find the real ones?

And besides, what we have done know is still just educated guesses, though they are very good guesses. For example, we guessed that the name of the init function is `init` - however, it could also just as well be something like `initWithObject`. Could we gain a little more certainty? It would be really convenient if there was something we could compare this against - something we know to be correct.

Well, luckily, there is: Android!

[Chapter 2.4: Comparing against Android](/handbook/vol2/chap2_4.md)

## Notes

> [Note 1] If there are multiple XREFs, it's likely that either the create function for the class has either been inlined, or there are multiple or them - RE to find out what is the case!

> [Note 2] The base `init` call is not necessarily a call to the actual base class's `init` for two reasons: inlining and the code not doing that. You can figure out what is the case by comparing against Android (explained in Chapter 3)
