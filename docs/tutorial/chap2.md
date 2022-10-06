# Chapter 2: Hooking & Patching

So we've got our own code running inside GD. Now we're faced with the biggest problem of all: how do we actually do stuff?

There are two fundamental tools in every GD modder's toolkit: **hooking** and **patching**. These are the building blocks from which we can start getting something interesting done.

In [Chapter 1](/docs/tutorial/chap1), it was stated that nowadays modders rarely work with binary code directly. There are, however, some cases in which working with raw binary code is in fact the optimal solution for some functionality in a mod. In these cases, it is done through a method called **patching**, which means applying, well, patches to binary code. Patches are, however, inherently **platform-dependent** and **unportable**, so their use is highly discouraged if higher-level options are available.

Patches do, however, still play a seminal role in GD modding. For example, one of the most famous mods, **noclip**, can be achieved with [a single binary patch](https://github.com/absoIute/Mega-Hack-v5/blob/master/bin/hacks/player.json#L7). There are also some very rare cases in complex mods where a single patch replaces writing hundreds of lines of C++ code. However, the keyword here and the key takeaway is "rare". Binary patches should never be your first solution to a problem, but when the time comes, don't be afraid to use them if they're clearly the best solution.

In contrast, **hooking** is not only more portable(-ish) but also arguably the most important tool in a GD modder's toolkit. To understand how it works, let's see some **code**:

```cpp
int addTwo(int a, int b) {
    return a + b;
}
```
Here we have a very simple **function** in C++: adding two numbers together. Let's suppose RobTop has added this function to GD, and we want to **hook it**.

The first thing you should know is that after compilation, each function will be given a **memory address**; that is, an address within GD's binary code where the function can be found. The second thing is that **in binary code, there are no names**. There are only memory addresses and values. Functions technically do not even exist in binary code; in there, they are called **subroutines**, and are referenced by their memory address.

This means that the function above gets compiled to something like this:
```cpp
int 0x482f4(int a, int b) {
    return a + b;
}
```
The memory addresses of functions are not constant between different compilations; this means that **a mod that works for GD version 2.113 does not work for version 2.112 or 2.2**, as the memory addresses of the functions are completely different. However, they are constant within copies of the same compiled binary, which means that if you find a function's address in the GD 2.113 binary, you can use that address in your mod and it will work for anyone with GD 2.113 installed.

It should also be noted however that function addresses are not actually constant, but their **offsets** are. The **base address** of GD is different every time you play the game, and this means that the **absolute memory addresses** of the functions are different too. However, the offsets to the base address are the same, so code like `gd_base_address + 0x1907b0` will always point to the same function (on a given version of GD). Getting the base address is also really simple; in Geode, this is done using the function `geode::base::get()`.


> This tutorial has been written by HJfod. If you find problems with it, please let me know on Discord at HJfod#1795.
