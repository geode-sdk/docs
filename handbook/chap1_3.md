# Chapter 1.3: Functions & Addresses

In [the last chapter](/handbook/chap1_2.md), we looked at hooking and how it works. However, the last chapter only touched hooking in theory. The code shown was not what actuals hooks in your code look like. For instance, we do not have access to GD's source code, so we can't exactly just write `return ourDetour()` at the start of the function we want to hook. Instead, we need to figure out some way to **insert hooks into GD's binary code**.

### MinHook

The traditional way of placing hooks used a library called [**MinHook**](https://github.com/TsudaKageyu/minhook). If you've ever developed GD mods or even used them yourself, you almost certainly have heard of MinHook before, or at least seen its 32-bit dynamic library `minhook.x32.dll`. However, using MinHook directly is **no longer considered good practice**. While it can work perfectly fine for a single mod, MinHook has a few issues: it's **Windows-only** and if you don't link to it as a dynamic library, it will cause **hook conflicts**.

Hook conflicts are a type of [**race condition**](https://en.m.wikipedia.org/wiki/Race_condition) and it happens when two mods try to hook the same function at the same time. If the mods do this sufficiently close to one another, there is a high chance that **one mod's hook will replace the other's**. The end result of this is that one of the mods functions incorrectly, when it fails to hook the function it expected to. In the best case, this just results in the mod losing functionality, but in the extreme case this **could cause crashes**.

?> This is not to say that MinHook is not a good library for hooking on Windows. It is actually quite good, as it's efficient and it in fact fixes hook conflicts if you link to it dynamically. However, this requires that **every mod** links MinHook dynamically, which is unfortunately not at all the case and also impossible to guarantee. However, **Geode actually uses MinHook under the hood** on Windows, as Geode is always linked dynamically, which solves the hook conflict problem.

But how does MinHook work? Let's examine some **real-world code** that uses it:
```cpp
bool (__thiscall *MenuLayer_init_o)(MenuLayer*);
bool __fastcall MenuLayer_init(MenuLayer* self) {
    if (!MenuLayer_init_o(self))
        return false;
    
    return true;
}

MH_CreateHook(
    reinterpret_cast<void*>(base + 0x1907b0),
    reinterpret_cast<void*>(&MenuLayer_init),
    reinterpret_cast<void**>(&MenuLayer_init_o)
);
```

Now, this code sure is quite a jump from the hooking code in the previous chapter. If you haven't done much low-level C++, you might be confused at a lot of the syntax here. There's a bit too much to take in from the code above, so for now we will just be concentrating on a few key details.

!> The above code contains things like **calling conventions** (`__thiscall` and `__fastcall`). Those will not be explained here as **Geode abstracts away calling conventions**. However, they will be explained in detail later on when talking about **reverse engineering**. The code also contains things like the `MenuLayer` type, which will be explained later when talking about GD types, and **function pointers**, which you can read more about [here](https://www.learncpp.com/cpp-tutorial/function-pointers/).

The most important part of this code is the line `base + 0x1907b0`. This is the **address** of the function. When C++ is compiled down to machine code, **all variable and function names are erased** and functions are instead given **memory addresses**. A memory address is just **the location in a binary that the function resides in**. For example, `base + 0x1907b0` means that the function is located `0x1907b0` (or 1640368 in decimal) bytes from the **base address** of the binary.

What this means is that in order to hook a function in GD, **we need to know its address**. On top of that, **we need to know its signature**, as your detour must always have the same signature as the function you're hooking.

### Addresses

So how do we find out these things? Usually, this is done through **reverse engineering**; however, RE is quite a complex skill and will be saved for later. And on top of that, **most common functions have already been found**. This means that instead of REing the function yourself, you can use **GD headers** that someone else has prepared that come with ready-made function signatures and addresses.

Traditionally, the most common GD header library was [**gd.h**](https://github.com/HJfod/gd.h). However, **Geode comes with its own set of GD headers**, as **gd.h is considered outdated**.

However, it is also important to note that **you still need to know how to reverse engineer** in order to make GD mods. Even if the addresses and signatures are all available, they still don't tell you what the function actually does, how it works, or where its called.

As noted previously, MinHook is no longer recommended to be used directly. Instead, **Geode comes with its own hooking syntax called `$modify`**. How it works will be explained in a later chapter, but first we must talk a bit about GD's game engine: **Cocos2d**.

[Chapter 1.4: Cocos2d](/handbook/chap1_4.md)
