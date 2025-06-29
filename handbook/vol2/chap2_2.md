# Chapter 2.2: Finding `MenuLayer::init`

The first thing we have to figure out when finding functions is exactly what function we want to find. Often times, this is some layer's `init` function - however, it can also be any function responsible for some behaviour we want to alter, like the function that is responsible for scaling objects, the function that plays a level's song, or anything else.

Luckily for us, **pretty much all functions in GD are part of some class**. This means that to find a function, we first have to make an educated guess about what class it might be in, and use that as a starting point for figuring out where it actually is.

If you look at the Symbol Tree window in Ghidra, you will see it has a folder named Classes. Opening this folder reveals, to great surprise, every class in GD:

![Picture showing the list of classes in GD open in Ghidra](/assets/handbook/vol2/classes_in_symbol_tree.png)

Now, there is one problem with this list: **there are a lot of classes in GD**. Just browsing this list is most likely not going to help you figure out where the function you're looking for is. Instead, first you need to make some guess about where to start.

For finding layers' `init` functions, this guess is very easy to make: just figure out the layer's name using DevTools, and it's probably in that class. 

So, to find the `init` function for `MenuLayer`, lets first navigate to `MenuLayer`:

![Picture showing MenuLayer's vtable open in Ghidra](/assets/handbook/vol2/MenuLayer_in_ghidra.png)

Unfortunately, there are no functions listed, as Ghidra can't really know which functions belong to which class. However, what there is listed are the class's **virtual function tables**; in other words, a list of all the virtual functions a class has. While not every class's `init` function is virtual, in `MenuLayer`'s case it is, so we can actually find the `init` function through the vtable directly.

Now, knowing exactly which index in the vtable is which virtual function is a bit difficult to figure out, but the jist is that virtual functions are in the order they were declared in the class itself. For `CCNode`, the first virtual function declared is `init` - however, as `CCNode` inherits from `CCObject` which has a bunch of a virtual functions of its own, `init` is not the first virtual function in the vtable, but instead the 10th.

If we look at the 10th function in `MenuLayer`'s vtable, we notice that it has a different name from the others:

![Picture showing MenuLayer's vtable open in Ghidra, focused on the index #9 which shows a function named FUN_005907b0 surrounded by other functions with human-readable names](/assets/handbook/vol2/MenuLayer_init.png)

The reason for this is that this function is the only one that's overridden - `MenuLayer` does not overwrite `setZOrder` for instance, so the function included in `MenuLayer`'s vtable is `CCNode::setZOrder`, whose name Ghidra knows as it is an exported symbol from `libcocos2d.dll`. However, because this function is overridden and defined in GD itself, Ghidra can't know its name. However, we know what its name is - it is `MenuLayer::init`!

![Picture showing MenuLayer::init's decompiled code open in Ghidra](/assets/handbook/vol2/MenuLayer_init_code.png)

Double-clicking the function name to enter it, we can look at its decompiled code. Since it uses sprites like `logo.png` which we can know is only used in the main menu by playing the game, we can verify that this is most certainly the function we're looking for. We can give it the correct name by selecting the function name and pressing L:

![Picture showing renaming MenuLayer::init in Ghidra](/assets/handbook/vol2/rename_MenuLayer_init.png)

Now the function also appears under the `MenuLayer` class in the class list, which is very convenient.

We can also fix the function's signature by right-clicking on it and selecting `Edit Function Signature`, setting its calling convention as `thiscall` and making it return a `bool`:

![Picture showing fixing MenuLayer::init's signature in Ghidra using the Edit Function Signature option](/assets/handbook/vol2/correct_MenuLayer_init_sig.png)

![Picture showing MenuLayer::init under MenuLayer in Ghidra](/assets/handbook/vol2/MenuLayer_init_in_list.png)

Congratulations, you have reverse-engineered your first function! However, let's not stop here - let's find some other, a little more obscure `init` functions:

[Chapter 2.3: Finding `LevelSettingsLayer::init`](/handbook/vol2/chap2_3.md)
