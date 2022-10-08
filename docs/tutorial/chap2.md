# Chapter 2: Hooking & Patching

So we've got our own code running inside GD. Now we're faced with a much bigger problem however; **how do we actually do stuff?**

There are two fundamental tools in every GD modder's toolkit: **hooking** and **patching**. These are the building blocks from which we can start getting something interesting done.

In [Chapter 1](/docs/tutorial/chap1), it was stated that nowadays modders rarely work with binary code directly. There are, however, some cases in which working with raw binary code is in fact the optimal solution for some functionality in a mod. In these cases, it is done through a method called **patching**, which means applying, well, patches to binary code. Patches are, however, inherently **platform-dependent** and **unportable**, so their use is highly discouraged if higher-level options are available.

Patches do, however, still play a seminal role in GD modding. For example, one of the most famous mods, **noclip**, can be achieved with [a single binary patch](https://github.com/absoIute/Mega-Hack-v5/blob/master/bin/hacks/player.json#L7). There are also some cases in complex mods where a few patches can replace writing hundreds of lines of C++ code. However, **it is very uncommon** for binary patches to be optimal. Binary patches should **never be your first solution to a problem**, but when the time comes, don't be afraid to use them if they're clearly the best solution.

In contrast, **hooking** is not only more portable(*-ish*) but also arguably the most important tool in a GD modder's toolkit. Understanding hooking is vitally important for modding, as it is most often the entrypoint from which a mod starts.

Consider the following function:

```cpp
int addTwo(int a, int b) {
    return a + b;
}
```

This function is quite simple; it is just adding two integers together. Let's say that this function is located in some other binary, and **we just want to know whenever it's called**, and do some stuff when that happens. **This is what hooking is for**; it lets you detect when a function is called, and do something with that information.

Hooking the function would look something like this:
```cpp
int addTwo(int a, int b) {
    addTwoHook(a, b);
    return a + b;
}

int addTwoHook(int a, int b) {
    std::cout << "addTwo called!\n";
}
```

A hook **always has the signature [[Note 1]](#Notes) as the function being hooked**. This means that we couldn't hook `addTwo` with something that takes two strings. Likewise, the return type has to be the same; if `addTwo` returns an `int`, so does our hook.

This is the basic premise of hooking: when the function you're hooking is called, the first thing you do is hop into your own code, and then hop back into the original once you're finished. The function that contains your own code is called a **detour**, and the funtion being hooked is called the **original**.

However, the code above is **actually misleading**. It would be more accurate to say that hooking does this:

```cpp
int addTwoOriginal(int a, int b) {
    return a + b;
}

int addTwo(int a, int b) {
    return addTwoDetour(a, b);
}

int addTwoDetour(int a, int b) {
    std::cout << "addTwo called!\n";
    return addTwoOriginal(a, b);
}
```

When you hook a function like `addTwo`, what first happens is that the body of the hooked function is stored somewhere else, and then the function body is **replaced** with a call to your detour [[Note 2]](#Notes). Then in your detour, you execute your own code and then return, either by calling the original or by giving your own return value.

Notice that you are not actually required to **call the original**, or return its value. We could just as easily make `addTwoDetour` do something completely different, for example like this:

```cpp
// never called!
int addTwoOriginal(int a, int b) {
    return a + b;
}

int addTwo(int a, int b) {
    return addTwoDetour(a, b);
}

int addTwoDetour(int a, int b) {
    std::cout << "addTwo called!\n";
    return a - b; // not calling the original
}
```

In this case, we skip calling the original function completely, and instead return the difference between `a` and `b`. Of course, this would be quite inconvenient to anyone calling `addTwo` who is expecting the numbers to be added together, but there's not much they could do about it; we have completely overwritten the function's definition.

We could also call the original, but pass it different parameters. Let's make `addTwo` act as normal, except always passing 7 as the second parameter and ignoring `b`:

```cpp
// this now always gets 7 as the b argument
int addTwoOriginal(int a, int b) {
    return a + b;
}

int addTwo(int a, int b) {
    return addTwoDetour(a, b);
}

int addTwoDetour(int a, int b) {
    std::cout << "addTwo called!\n";
    return addTwoOriginal(a, 7);
}
```

Alternatively, let's say we want to use the result of `addTwo` ourselves. We can do this by utilizing it just like any other function call:

```cpp
int addTwoOriginal(int a, int b) {
    return a + b;
}

int addTwo(int a, int b) {
    return addTwoDetour(a, b);
}

int addTwoDetour(int a, int b) {
    int res = addTwoOriginal(a, b);
    std::cout << "addTwo returned " << res << "\n";
    return res;
}
```

Here, we first call the original to see its result, then log it into the console and then return the result as normal. We could also just return something else:

```cpp
int addTwoOriginal(int a, int b) {
    return a + b;
}

int addTwo(int a, int b) {
    return addTwoDetour(a, b);
}

int addTwoDetour(int a, int b) {
    int res = addTwoOriginal(a, b);
    std::cout << "addTwo returned " << res << "\n";
    return 0;
}
```

In this case, we can use the result of the original `addTwoOriginal` function as we please, however any callers of `addTwo` will always get 0 as a result.

At this point, hooking might still feel difficult to wrap your head around. Don't worry though; hooking will be a seminal part of this whole handbook, and **you will see plenty more practical examples of it going forward**.

However, at this point it should be noted that **the syntax for hooking in Geode looks quite different** from what was shown here. The underlying premise, however, is always the same: you replace the function body with a call to your own, and then do whatever you want. You may call the original function and use its result however you'd like, or you may completely overwrite the function's behaviour by not calling the original at all.

For now, we can leave hooking be, as before we can find any practical applications for it, we must first **find some functions to hook**.

[Chapter 3: Functions & Addresses](/docs/tutorial/chap3.md)

### Notes

> [Note 1] **Signature** means the parameter and return types of a function, i.e. `int addTwo(int, int)`.

> [Note 2] This, too, is not actually how hooking is implemented most of the time. What actually is the case is that **only the first few statements** of a function are replaced by the call to the detour, and then the call to the original function does a bunch of jumps and redirects to execute the whole original body. For the purposes of this tutorial however, **it is easier to think of the whole function body as being replaced** instead of just a part of it.
