# Memory Management

Managing memory in general with C++ has been partially covered in [C++ tips](/cpp/tips.md), but Cocos2d comes with its own memory management system that is all modders have to learn at some point. On top of this, Geode adds two classes, [`Ref`](/classes/geode/Ref) and [`WeakRef`](/classes/geode/WeakRef), which Geode devs will tell you to use but not how to use them. This tutorial aims to remedy that, explaining how memory management in Cocos2d works in detail.

## Memory management

Cocos2d's memory management features are available to all classes that inherit from [`CCObject`](/classes/cocos2d/CCObject), as it is the base class that implements the required features. Cocos2d's memory management works by **ref counting** - in other words, it just stores how many references exist to an object, and once the ref count reaches 0, the object's memory is freed.

Unfortunately, due to how C++ works, the ref counting is not directly automatic - it doesn't know where the object is actually used, only the ref count stored inside it. In practice, classes like [`CCNode`](/classes/cocos2d/CCNode) always handle retaining and releasing their children, so you usually don't have to deal with memory management when building nodes. However, this does have consequences when it comes to class members, as we will see later.

The easiest way to concretize this is to look at **manual memory management** in Cocos2d, which works by using the [`retain`](/classes/cocos2d/CCObject#retain) and [`release`](/classes/cocos2d/CCObject#release) functions. In their simplicity, `retain` increments the object's ref count by one, and `release` decrements it by one.

```cpp
// Ref count of a created object is always 1
auto node = CCNode::create();

// Increment ref count to 2
node->retain();

// Decrement ref count to 1, and then to 0, freeing the object
node->release();
node->release();
```

You can think of `retain` and `release` like the following: `retain` tells Cocos2d "Hey, I'm still using this object, please don't free it!", and `release` tells it "Okay, I don't need this object anymore!"

In practice, if you have to do manual memory management, you likely won't be using the `retain` and `release` functions directly and instead use the `CC_SAFE_RETAIN` and `CC_SAFE_RELEASE` macros:

```cpp
// Increment ref count
CC_SAFE_RETAIN(node);
// Decrement ref count
CC_SAFE_RELEASE(node);
```

The difference these macros have over direct function calls is that they check that the object isn't `nullptr` first.

> :warning: You should never call `release` more than the amount of times you have called `retain` on an object to force-free it, as if some other code holds a pointer to the object and is expecting it to be retained, you will cause that code to break.

## Classes

Remembering to manage every object you have ever created is a real problem, though. Luckily, most Cocos2d classes call `retain` for you.

```cpp
auto node = CCNode::create();
auto other = CCNode::create();
// CCNode increments other's ref count by 1, and decrements it when the child
// is removed or when node is freed
node->addChild(other);

auto array = CCArray::create();
// CCArray increments node's ref count by 1, and decrements it when the node
// is removed or when the array is freed
array->addObject(node);
```

This means that if you store your created object as a child to something or add it to an array, you don't have to worry about memory management. However, if you don't do that, you need to be careful:

```cpp
{
    auto array = CCArray::create();
    // If this array is never used anywhere, its ref count stays at 1 and
    // nothing calls release on it, so the memory is leaked - except it's not!
}
```

This code looks like it should cause a memory leak, since nothing ever calls `release` on the array. However, Cocos2d has an **automatic garbage collector** that handles situations like this. You may have seen that all `create` functions in nodes call a function called `autorelease`:

```cpp
SomeNode* SomeNode::create() {
    auto ret = new SomeNode();
    if (ret->init()) {
        // Make the node be automatically garbage collected
        ret->autorelease();
        return ret;
    }

    delete ret;
    return nullptr;
}
```

This tells Cocos2d that if no calls to `retain` are made to this object, it should be freed automatically later on. This means that the following code:

```cpp
{
    auto array = CCArray::create();
}
```

does not cause a memory leak, as Cocos2d notices no `retain` calls to `array` have been made, and frees it.

However, the array is not freed immediately; instead, it is usually freed at earliest on the next frame. This means that you can still use the array to do things:

```cpp
auto array = CCArray::create();
array->addObject(...);
someNode->someFunction(array);
```

and it will be automatically freed later on.

## Class members

`autorelease` is very neat when you just want to quickly create an array to pass into a function, or something similar. However, it causes a bit of a headache when dealing with class members:

```cpp
class MyNode : public CCNode {
protected:
    CCArray* m_array;

    bool init() {
        if (!CCNode::init())
            return false;

        m_array = CCArray::create();

        return true;
    }

public:
    CCObject* getFirst() const {
        return m_array->firstObject();
    }
};
```

If you ran this code, you would notice that your mod mysteriously crashes in any call made to `getFirst` after a while. The reason for this is that `m_array` has been freed - no calls to `retain` were ever made to it, and since **Cocos2d ref counting doesn't actually have any knowledge of what pointers to an object exist**, Cocos2d concludes that the array must not be in use.

The solution, then, is to call `retain` on the array:

```cpp
class MyNode : public CCNode {
protected:
    CCArray* m_array;

    bool init() {
        if (!CCNode::init())
            return false;

        m_array = CCArray::create();
        m_array->retain();

        return true;
    }

public:
    CCObject* getFirst() const {
        return m_array->firstObject();
    }
};
```

Now Cocos2d knows that `m_array` is actually being used, and won't free it. However, now we have caused a memory leak: when `MyNode` is destroyed, the array's ref count stays at 1, meaning it never gets freed. To fix this, we need to add a call to `release` in `MyNode`'s destructor:

```cpp
class MyNode : public CCNode {
protected:
    CCArray* m_array;

    bool init() {
        if (!CCNode::init())
            return false;

        m_array = CCArray::create();
        m_array->retain();

        return true;
    }

    virtual ~MyNode() {
        m_array->release();
    }

public:
    CCObject* getFirst() const {
        return m_array->firstObject();
    }
};
```

This is the conventional pattern for working with `CCObject` members, and you may find yourself reading old code that uses it, or even using it yourself. However, it comes with a problem: **this gets really complex to work with really fast**. Imagine that `m_array` is not just a member that is created once at the start and freed at the end, but instead may be `nullptr`, or may be removed later, or may even be swapped with other arrays. Dealing with ref counting manually in that case is a real pain:

```cpp
class MyNode : public CCNode {
protected:
    CCArray* m_array = nullptr;

    bool init() {
        if (!CCNode::init())
            return false;

        m_array = CCArray::create();
        m_array->retain();

        return true;
    }

    virtual ~MyNode() {
        CC_SAFE_RELEASE(m_array);
    }

public:
    void swapArray(CCArray* other) {
        // Release existing array
        CC_SAFE_RELEASE(m_array);
        // Swap array
        m_array = other;
        // Retain the new array
        CC_SAFE_RETAIN(m_array);
    }

    void removeArray() {
        CC_SAFE_RELEASE(m_array);
        m_array = nullptr;
    }
};
```

When you add in more members, trying to keep track of all these relations gets really complex, and the possibility of an accidental memory leak / memory issue increases exponentially. Luckily, Geode has a solution: [`Ref`](/classes/geode/Ref).

## Ref

`Ref` is a **smart pointer** for `CCObject`s - in essence, it's just a class that retains the object it points to, and releases it when `Ref` goes out of scope. In other words, it's a [RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization) alternative to manual `retain` and `release` calls. Using it, we can refactor our previous code to just this:

```cpp
class MyNode : public CCNode {
protected:
    Ref<CCArray> m_array = nullptr;

    bool init() {
        if (!CCNode::init())
            return false;

        m_array = CCArray::create();

        return true;
    }

public:
    void swapArray(CCArray* other) {
        // Swap array
        m_array = other;
    }

    void removeArray() {
        m_array = nullptr;
    }
};
```

Notice that all manual calls to `retain` and `release` disappeared from the code - `Ref` handles all of them for you. This makes writing and reasoning about code much simpler - you can be assured that as long as you have initially made a `Ref` to a valid object, it's always going to stay valid, and the memory will be freed appropriately when you actually no longer use it.

`Ref` is also relatively cheap - if you are unsure whether the lifetime of a pointer you have extends to your usage, just store the pointer in a `Ref` to stay safe.

**In general, you should at least stick `Ref` to all your class members, unless you can be certain about the lifetime of the object otherwise.**

## WeakRef

`Ref` does have a problem, however: since it increments the ref count, the object being pointed to will only be freed once the `Ref` goes out of scope. However, sometimes this won't happen at a desirable time: for example, some mod might have a map like the following:

```cpp
static std::unordered_map<CCNode*, SomeData> REGISTERED_NODES {};

void addDataToNode(CCNode* node, SomeData const& data) {
    REGISTERED_NODES.insert({ node, data });
}

void doSomethingWithNodes() {
    for (auto& [node, data] : REGISTERED_NODES) {
        // ...
    }
}
```

For example, a right mouse click API might store the nodes that have registered themselves as mouse right click event listeners like this. However, this code is never told when the `CCNode` pointer it has is no longer valid, so if the scene is changed and the nodes are freed while they are still in `REGISTERED_NODES`, the next call to `doSomethingWithNodes` will cause a crash trying to access already freed memory.

Unfortunately, C++ has no way to know if a raw pointer is valid or not. Your first instinct here might be to make the `CCNode` a `Ref`:

```cpp
static std::unordered_map<Ref<CCNode>, SomeData> REGISTERED_NODES {};

void addDataToNode(CCNode* node, SomeData const& data) {
    REGISTERED_NODES.insert({ node, data });
}

void doSomethingWithNodes() {
    for (auto& [node, data] : REGISTERED_NODES) {
        // ...
    }
}
```

Now you can be assured that `REGISTERED_NODES` only contains valid pointers to `CCNode`s. However, now we have a memory leak: if the user switches scenes, the only reference left to the `CCNode` will be the one in `REGISTERED_NODES`, which is probably undesirable, since we'd want the node to be removed from `REGISTERED_NODES` too when the node is no longer visible.

A primitive solution to fixing this would be to check the node's ref count, and if it's 1 then we know only `REGISTERED_NODES` has a reference to it:

```cpp
static std::unordered_map<Ref<CCNode>, SomeData> REGISTERED_NODES {};

void freeUnusedNodes() {
    for (auto& [node, _] : REGISTERED_NODES) {
        if (node->retainCount() == 1) {
            REGISTERED_NODES.erase(node);
        }
    }
}

void addDataToNode(CCNode* node, SomeData const& data) {
    REGISTERED_NODES.insert({ node, data });
}

void doSomethingWithNodes() {
    freeUnusedNodes();
    for (auto& [node, data] : REGISTERED_NODES) {
        // ...
    }
}
```

However, this plan immediately falls apart once some other mod does the same, as the ref count is never reaching 1.

For this reason, Geode provides another class: `WeakRef`, which is like `Ref` but it doesn't change the ref count:

```cpp
static std::unordered_map<WeakRef<CCNode>, SomeData> REGISTERED_NODES {};

void addDataToNode(CCNode* node, SomeData const& data) {
    REGISTERED_NODES.insert({ node, data });
}

void doSomethingWithNodes() {
    for (auto& [node, data] : REGISTERED_NODES) {
        // ...
    }
}
```

The difference between `Ref` and `WeakRef` is that `Ref` allows you to access the pointed object directly since it's guaranteed to be valid. However, `WeakRef` has no such guarantees since it doesn't impact the ref count, so you need to first lock it to see if the object is still valid:

```cpp
auto ref = WeakRef(obj);

// Later on
// Check if the reference still points to a valid object
if (auto obj = ref.lock()) {
}
```

`lock` returns a `Ref`, so you have guaranteed safe access to the object for as long as you have the `Ref`. As `WeakRef` does not increment the reference count, if something frees the object beforehand, `lock` will return a null `Ref`.

Now, `WeakRef` still won't automatically remove itself from your maps, so you do still have to manually clear invalid pointers:

```cpp
static std::unordered_map<WeakRef<CCNode>, SomeData> REGISTERED_NODES {};

void freeUnusedNodes() {
    for (auto& [node, _] : REGISTERED_NODES) {
        if (node.lock() == nullptr) {
            REGISTERED_NODES.erase(node);
        }
    }
}

void addDataToNode(CCNode* node, SomeData const& data) {
    REGISTERED_NODES.insert({ node, data });
}

void doSomethingWithNodes() {
    freeUnusedNodes();
    for (auto& [node, data] : REGISTERED_NODES) {
        // ...
    }
}
```

However, now you can assured that this scheme won't break if some other mod does the same.

> :warning: Note that a node that has weak references to it is only freed once one of those weak references triggers a recheck of the object. This is because WeakRef internally does actually increment the reference count, and uses the "check if reference count is 1" trick to know that only weak references exist to an object. This might seem counter-productive, but that scheme does work if Geode's WeakRef system is the only one using it at a time, which it should be if modders follow guidelines properly.
