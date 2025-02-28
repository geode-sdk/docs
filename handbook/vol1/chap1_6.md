# Chapter 1.6: Modifying Layers

So we know how to figure out the names of layers, but how do we actually modify them and add our own stuff?

If we want to modify what another node does, we must remember how hooking works: via hooking, we can modify **functions**. This is great, but unfortunately, classes aren't functions, so we can't exactly just "hook a layer". Instead, let's break down our problem a bit.

Layers are **instances** of a specific class. Them being instances means that even if we manage to modify one instance, if the user encounters another instance of the same class, we will have to be able to modify that aswell. Extrapolating from that, **our goal is to modify every instance of a specific class**.

Since hooking is only possible on functions, what we want to do is **find some function that is called for every instance of that class**. In particular, we want something that is as close to the creation of that instance as possible. If we add our own stuff with a delay, that would cause an inconvenient user experience - buttons should be there when the user enters a layer, not 5 seconds after! The function should also **only be called once per instance**, since if our modification involves adding a button for example, we don't want to add multiple copies of the same button.

Luckily for us, the **design patterns of Cocos2d nodes** provides us with a perfect candidate for nearly all classes.

## The Structure of a Node

Let's look at **the general structure of a node**:

```cpp
class SomeNode : public CCNode {
protected:
    bool init() {
        if (!CCNode::init())
            return false;

        // Initialize SomeNode

        return true;
    }

public:
    static SomeNode* create() {
        auto ret = new SomeNode();
        if (ret->init()) {
            ret->autorelease();
            return ret;
        }

        delete ret;
        return nullptr;
    }
};
```

Every node, and as such layer, has at least two functions: `create` and `init` [[Note 1]](#notes). `create`, as explained [in the previous chapter](/handbook/vol1/chap1_5.md), is what you use to create instances of the class.

What we're interested in right now however is `init`. This is the function where the node initializes itself; adds all of its subnodes, sets its delegates, etc.. As we can see in the definition of `create`, every instance of `SomeNode` is first created, and then its `init` function is invoked. On top of this, the `init` function is only called once per node, as it doesn't make sense to initialize the same node multiple times.

This should start to sound familiar: this is exactly what we outlined to be looking for at the start of the chapter!

Although, an observant reader may be wondering right now; if our goal is to find a function that is called every time a node is created, and is only called once per node, **why don't we just hook that layer's `create` function?** This is a good question; and the explanation is two-fold.

Firstly, `create` returns the created node, whereas in `init` we have direct access to the node through the `this` parameter (as `init` is a member function, and not static). This is only a code convenience issue however.

The bigger issue with hooking `create` is due to a problem known as **inlining**. We can't get into the depths of what that means right now, but in practice what inling means is that **some functions just aren't hookable**, and this includes many classes' `create` functions. In contrast, only a few classes' `init` functions are inlined, so we can be much more confident we can actually make the hook if we target `init`.

On top of this, hooking `init` actually brings us quite close to the actual design pattern Cocos2d uses, although to see that, we need to first hook an init function, which brings us to `$modify`.

## `$modify`

One of the most important concepts in all of Geode is the `$modify` macro; it is Geode's **syntactically sugary hooking system**. What it looks like is this:

```cpp
class $modify(ClassName) {
    // Overwrite functions, etc.
};
```

You will find that this declaration closely resembles a class, and underlyingly it in fact is one; however, **modify should not be treated like a normal class**. `$modify` comes with extra superpowers and **limitations** specific to hooking functions. Something that works for normal C++ classes is not guaranteed to work the same in `$modify`, even if the compiler might let you do it.

The name and syntax of `$modify` comes from its purpose; it is to **modify classes**.

For example, to modify the main menu, it would look like this (remembering from [the previous chapter](/handbook/vol1/chap1_5.md) that the main menu's layer is called `MenuLayer`):

```cpp
#include <Geode/modify/MenuLayer.hpp>

class $modify(MenuLayer) {
    // Overwrite functions, etc.
};
```

For the sake of compile-time performance, Geode's modifiers are split into files, which you can include by simply including `Geode/modify/{Name of the class}.hpp`. **You can only use `$modify` on classes that have been included**; in practice, this includes nearly all GD classes but does not include any of your own classes or Geode's classes.

You will sometimes see `$modify` be used with two arguments; this is to provide a **name** for the modified class, which is useful in many common situations.

```cpp
#include <Geode/modify/MenuLayer.hpp>

class $modify(MyModifiedMenuLayer, MenuLayer) {
    // You can now refer to the modified class as 'MyModifiedMenuLayer'
};
```

If you don't provide a name, Geode will automatically generate a random name for the class, which is meant to not cause any name collisions.

> :warning: As `$modify` does not create a normal class, you should not expect standard C++ class things like adding members to work. Geode does come with [an utility for adding members to classes](/tutorials/fields.md), which is quite close to the normal way of declaring members, but not exactly the same.

## Hooking `init`

Ok, we know how to start modifying a class, but this doesn't yet do what we want. How exactly are supposed to hook `init`?

This is where the real magic of `$modify` comes in: to hook `MenuLayer::init`, all we have to do is **redefine the function in our modified class**:

```cpp
#include <Geode/modify/MenuLayer.hpp>

class $modify(MenuLayer) {
    // Signature is `bool MenuLayer::init()`
    bool init() {
        // Our code!
    }
};
```

That's it! Now if you compiled this mod, installed it and opened the game, you would find that `MenuLayer` has turned completely blank, as we have overridden its `init` function, which means it can't add any of its nodes. You would also most likely find that the game would crash, since a lot of things depend on `MenuLayer` actually containing stuff.

And, well, we don't want to _override_ MenuLayer; we just want to append stuff to it. Luckily, from [the hooking chapter](/handbook/vol1/chap1_2.md), we know there is a tool for this: just **call the original function**!

...well, uh. How exactly do we do that?

The answer is simple: we, uh, call the original:

```cpp
#include <Geode/modify/MenuLayer.hpp>

class $modify(MenuLayer) {
    // Signature is `bool MenuLayer::init()`
    bool init() {
        if (!MenuLayer::init())
            return false;

        // Our code!

        return true;
    }
};
```

That's it. As previously stated, this is [quite close to the standard node design pattern](#the-structure-of-a-node).

> :warning: It should be noted that not all layer's `init` functions have the same signature, and `$modify` requires the signature to **exactly match** in order to create a hook. Unfortunately, due to implementation problems, it also (currently) doesn't tell you if your signature is wrong, so you may find yourself scratching your head as to why your mod isn't working, only to realize the signature of `init` is off. Check [GeometryDash.bro](https://github.com/geode-sdk/bindings/blob/main/bindings/2.2074/GeometryDash.bro) to make sure the signature of your hook is correct!

Now we are at an interesting point; we know how to hook functions, we know what function from a layer to hook in order to modify it, and we know how to work with nodes. So, let's tie all of this together! Only 7 chapters in, [it's time for **Hello, World!**](/handbook/vol1/chap1_7.md)

## Notes

> [Note 1] There are some very rare cases of nodes that don't have an `init` or `create` function. However, for the purposes of this tutorial, we will pretend those don't exist.

