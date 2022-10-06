# Chapter 2: Hooking & Patching

So we've got our own code running inside GD. Now we're faced with the biggest problem of all: how do we actually do stuff?

There are two fundamental tools in every GD modder's toolkit: **hooking** and **patching**. These are the building blocks from which we can start getting something interesting done.

In [Chapter 1](/docs/tutorial/chap1), it was stated that nowadays modders rarely work with binary code directly. There are, however, some cases in which working with raw binary code is in fact the optimal solution for some functionality in a mod. In these cases, it is done through a method called **patching**, which means applying, well, patches to binary code. Patches are, however, inherently **platform-dependent** and **unportable**, so their use is highly discouraged if higher-level options are available.

Patches do, however, still play a seminal role in GD modding. For example, one of the most famous mods, **noclip**, can be achieved with [a single binary patch](https://github.com/absoIute/Mega-Hack-v5/blob/master/bin/hacks/player.json#L7). There are also some very rare cases in complex mods where a single patch replaces writing hundreds of lines of C++ code. However, the keyword here and the key takeaway is "__rare__". Binary patches should **never be your first solution to a problem**, but when the time comes, don't be afraid to use them if they're clearly the best solution.

In contrast, **hooking** is not only more portable(*-ish*) but also arguably the most important tool in a GD modder's toolkit.


