# Chapter 2.4: Comparing Against Android

Android GD is special for one important reason - **it has symbols!** What this means is that every function name is present in the code, just like Cocos2d on Windows - and not only their names, but their parameter types aswell! And because GD is a cross-compiled game from a single codebase, this means that we can compare against Android GD to aid reverse engineering other platforms.

Android GD's binary is called `libcocos2dcpp.so` - it contains GD, Cocos2d, and all other dependencies in a single package. However, getting your hands into it is not as easy as Windows, since you need to extract it from the APK - ask some other modder for their copy of it :wink:

> :warning: ARMv8 support in Ghidra broke in version 11, so you'd preferably get the ARMv7 version.

Once you have acquired the Android binary, import it to Ghidra the same way you imported Windows, except make sure to disable the `Non returning functions: discovered` option for Analysis.

![Android GD imported to Ghidra](/assets/handbook/vol2/add_android_to_ghidra.png)

![Analyzing Android GD, with the `Non returning functions: discovered` option disabled](/assets/handbook/vol2/Android_analyz.png)

So, let's verify the REing we did [in the last chapter](/handbook/vol2/chap2_3.md)!

Open up the class list on Android and look up `LevelSettingsLayer`:

![`LevelSettingsLayer` from Android open in Ghidra, showing a lot more functions](/assets/handbook/vol2/LevelSettingsLayer_androir.png)

Whoa, that's a lot of functions... thanks, symbols! The first function we found in the last chapter was the constructor, so let's navigate straight to it!

Well, hm. There doesn't seem to be any function named `LevelSettingsLayer` here? The destructors are there but no constructor? Suspicious... let's open up `create` instead to see what's up:

![`LevelSettingsLayer::create` on Android, showing that the constructor has been inlined](/assets/handbook/vol2/LevelSettingsLayer_create_androir.png)

Ah, so that's what happened - **the constructor got inlined**. This means that its code was inserted to its call sites and the function definition itself got removed.

Well, if we look at the end of the `create` function, we can see the call to `init` is there:

![`LevelSettingsLayer::init` called inside `LevelSettingsLayer::create`](/assets/handbook/vol2/LevelSettingsLayer_create_call_to_init_androir.png)

Let's follow the call and compare this to the `LevelSettingsLayer::init` we found on Windows:

![Comparison between `LevelSettingsLayer::init` on Android and what we think is the same on Windows](/assets/handbook/vol2/compare_LSL_inits.png)

(Left is Android, right is Windows)

Hmm, alright, both have the call to `FLAlertLayer::init` at the start. And both make calls to `CCObject::retain`, `CCDirector::sharedDirector`, and the `CCRect` constructor at similar times... Oh, and there's a string literal! Both have a call to `CCLabelBMFont::create` with the same parameters. String literals are a really good way to compare functions, so this confirms it - we definitely found the real `LevelSettingsLayer::init` on Windows.

So, let's copy the signature from Android to Windows:

![The signature of `LevelSettingsLayer::init` on Android, acquired through a symbol](/assets/handbook/vol2/LevelSettingsLayer_init_sig.png)

Note that the Android function signatures aren't perfect - they don't have parameter names, and more crucially they **don't have a return type**. You may notice that Ghidra has *inferred* the type of `LevelSettingsLayer::init` to be a void, however we know better than Ghidra that `init` functions don't return void! Manually fixing the return type makes Ghidra add the missing `return false`:

![`LevelSettingsLayer::init` on Android in Ghidra, now returning the correct type](/assets/handbook/vol2/androir_fixed_init_ret_type.png)

So let's get back to adding the correct parameter types on Windows. The first one is a `LevelSettingsObject*`, so let's add it-

![The type `LevelSettingsObject` doesn't exist in Ghidra](/assets/handbook/vol2/LevelSettingsObject_invalid.png)

Oh, Ghidra has no clue what a `LevelSettingsObject` is. Well, we could manually define the type using the Data Type Manager, but it's better to let Ghidra make the definition for us by finding some `LevelSettingsObject` member function and typing it as `__thiscall`:

![`LevelSettingsObject::init` renamed correctly, which causes Ghidra to register the `LevelSettingsObject` type](/assets/handbook/vol2/LevelSettingsObject_init.png)

(`LevelSettingsObject::init` was found using the same method as the previous chapters, however replicating it is left as an exercise for the reader!)

Repeat the same for `LevelEditorLayer`, and now Ghidra lets us use them as parameter types:

![`LevelSettingsLayer::init` with the correct parameter types on Windows](/assets/handbook/vol2/LevelSettingsLayer_init_retyped.png)

> :warning: The reason the **calling convention** of `LevelSettingsLayer::init` is `__thiscall` will be explained in a later chapter - not all functions have the same calling convention!

And there we go - we can now be pretty much certain that we have found the correct `LevelSettingsLayer::init`, and that our signature is right!

[Chapter 2.5: Finding Callbacks](/handbook/vol2/chap2_5.md)
