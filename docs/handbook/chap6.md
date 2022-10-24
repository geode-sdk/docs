# Chapter 6: Modifying Layers

So we know how to figure out the names of layers, but how do we actually modify them and add our own stuff?

Before we can start modifying layers, we should understand **the general structure of a `CCNode`**:

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
        if (ret && ret->init()) {
            ret->autorelease();
            return ret;
        }
        CC_SAFE_DELETE(ret);
        return nullptr;
    }
};
```

Every node, and as such layer, has at least two functions: `create` and `init` [[Note 1]](#notes). `create`, as explained [in the previous chapter](/docs/handbook/chap5.md), is what you use to create instances of the class.

What we're interested in right now however is `init`. This is the function where the node initializes itself; adds all of its subnodes

### Notes

> [Note 1] There are some very rare cases of nodes that don't have an `init` or `create` function. However, for the purposes of this tutorial, we will pretend those don't exist.

