# Chapter 1.3: Functions & Addresses

In [the last chapter](/handbook/vol1/chap1_2.md), we looked at hooking and how it works. However, the last chapter only touched hooking in theory. The code shown was not what actuals hooks in your code look like. For instance, we do not have access to GD's source code, so we can't exactly just write `return ourDetour()` at the start of the function we want to hook. Instead, we need to figure out some way to **insert hooks into GD's binary code**.

## Manual hooking

The main way to create hooks in Geode is using an abstraction called `$modify` - however, it is a very powerful tool and hard to explain without some preface. It is also just an abstraction; you can create hooks manually in Geode as well through the [`Mod::addHook`](/classes/geode/Mod#addHook) interface - although in practice **you should never be creating manual hooks unless necessary**.

> :information_source: Manual hooks are sometimes necessary, for example to hook some obscure low-level functions that only exists on one platform, like GLFW on Windows. However, for 99% of mods, you should just be using `$modify`, since it makes your code much more portable and easier to work with.

> :information_source: The traditional way in 2.1 of placing hooks used a library called [**MinHook**](https://github.com/TsudaKageyu/minhook). If you've been in the GD modding scene before 2.2, you've almost certainly heard of MinHook before, or at least seen its 32-bit dynamic library `minhook.x32.dll`. However, using MinHook directly is **no longer considered good practice**. While it can work perfectly fine for a single mod, MinHook has a few issues: it's Windows-only. hard to use, and if you don't link to it as a dynamic library, it will cause **hook conflicts** [[Note 1]](#notes).

Let's see a [real-world example](/tutorials/manualhooks) of creating manual hooks in Geode:
```cpp
auto wrapFunction(uintptr_t address, tulip::hook::WrapperMetadata const& metadata) {
	auto wrapped = geode::hook::createWrapper(reinterpret_cast<void*>(address), metadata);
	if (wrapped.isErr()) {{
		throw std::runtime_error(wrapped.unwrapErr());
	}}
	return wrapped.unwrap();
}

void MenuLayer_onNewgrounds(MenuLayer* self, CCObject* sender) {
    log::info("Hook reached!");
	static auto original = wrapFunction(
        geode::base::get() + 0x27b480,
        tulip::hook::WrapperMetadata{
            .m_convention = geode::hook::createConvention(tulip::hook::TulipConvention::Thiscall),
            .m_abstract = tulip::hook::AbstractFunction::from(void(*)(MenuLayer*, CCObject*)),
        }
    );
    reinterpret_cast<void(*)(MenuLayer*, CCObject*)>(original)(self, sender);
    log::info("After original!");
}

$execute {
    Mod::get()->addHook(
        reinterpret_cast<void*>(geode::base::get() + 0x27b480),
        &MenuLayer_onNewgrounds,
        "MenuLayer::onNewgrounds",
        tulip::hook::TulipConvention::Thiscall
    );
}
```

Now, this code sure is quite a jump from the hooking code in the previous chapter. If you haven't done much low-level C++, you might be confused at a lot of the syntax here. There's a bit too much to take in from the code above, so for now we will just be concentrating on a few key details.

The most important part of this code is `geode::base::get() + 0x27b480`. This is the **address** of the function. When C++ is compiled down to machine code, **all variable and function names are erased** and functions are instead given **memory addresses**. A memory address is just **the location in a binary that the function resides in**. For example, `geode::base::get() + 0x27b480` means that the function is located at offset `0x27b480` (or 2602112 in decimal) bytes from the **base address** - GD's base address, given by the function `geode::base::get()`.

> :information_source: The reason we need to add the base address to the function's address is because **the base address of GD is dynamic** - it changes between startups!

What this means is that in order to hook a function in GD, **we need to know its address**. On top of that, **we need to know its signature**, as your detour must always have the same signature as the function you're hooking - otherwise the game will crash!

> :warning: The above code snippet also references **calling conventions** - if you use `$modify`, Geode handles them for you, however if you're doing manual hooks or reverse engineering, these are very important to get right.

## Addresses

So how do we find out these things? Usually, this is done through **reverse engineering**; however, RE is quite a complex skill, and would take far too much time to explain here, so it has its [own dedicated volume instead](https://docs.geode-sdk.org/handbook/vol2/chap2_1). And on top of that, **most common functions have already been found**. This means that instead of REing the function yourself, you can use the GD bindings that come packaged with Geode.

> :information_source: Traditionally, the most common GD header library was [**gd.h**](https://github.com/HJfod/gd.h). However, nowadays **gd.h is completely obsolete**, as it is only for 2.1 and fully unmaintained.

However, it is also important to note that **you still need to know how to reverse engineer** in order to make GD mods. Even if the addresses and signatures are all available, they still don't tell you what the function actually does, how it works, or where its called.

As noted previously, you shouldn't usually be creating hooks manually. Instead, **Geode comes with a special hooking syntax called `$modify`**. How it works will be explained in a later chapter, but first we must talk a bit about GD's game engine: **Cocos2d**.

[Chapter 1.4: Cocos2d](/handbook/vol1/chap1_4.md)

## Notes

> [Note 1] Hook conflicts are a type of [**race condition**](https://en.m.wikipedia.org/wiki/Race_condition) and it happens when two mods try to hook the same function at the same time. If the mods do this sufficiently close to one another, there is a high chance that **one mod's hook will replace the other's**. The end result of this is that one of the mods functions incorrectly, when it fails to hook the function it expected to. In the best case, this just results in the mod losing functionality, but in the extreme case this **could cause crashes**.

