# Migrating from Geode v4.x to v5.0

## Changes to `Popup`

`geode::Popup` is no longer templated, and instead accepts its own arguments in `Popup::init` (which also replaces `initAnchored`). It now uses the pattern similar to any other node. For example the following code:

```cpp
class MyPopup : public Popup<int> {
public:
    static MyPopup* create(int value) {
        auto popup = new MyPopup;
        if (popup->initAnchored(320.f, 160.f, value)) {
            popup->autorelease();
            return ret;
        }
        delete ret;
        return nullptr;
    }

protected:
    bool setup(int value) override {
        auto node = CCLabelBMFont::create("Hi", "bigFont.fnt");
        this->addChild(node);

        return true;
    }
};
```

Should be refactored to this:

```cpp
class MyPopup : public Popup {
public:
    static MyPopup* create(int value) {
        auto popup = new MyPopup;
        if (popup->init(value)) {
            popup->autorelease();
            return ret;
        }
        delete ret;
        return nullptr;
    }

protected:
    bool init(int value) {
        if (!Popup::init(320.f, 160.f))
            return false;

        auto node = CCLabelBMFont::create("Hi", "bigFont.fnt");
        this->addChild(node);

        return true;
    }
};
```

## Changes to `std::function` arguments

Almost all parts of geode that accepted a `std::function` (such as `queueInMainThread`, `TextInput::setCallback`, etc.) have been changed to use `geode::Function` instead. The primary difference is that a `geode::Function` cannot be copied, only moved. Most proper usages should still compile and work properly, but if you have errors that are related to `std::function`, try adding `std::move`s or making sure to explicitly use `geode::Function` everywhere.

## Changes to string arguments

A ton of functions that accepted `std::string_view`, `std::string` or `std::string const&` have been written uncarefully and inefficiently before, which v5 aimed to fix. For many cases, you don't need to change anything, but conversion between string types isn't always possible, so this might break some code. This applies to return values as well, certain commonly used functions like `CCNode::getID` have been changed to return `geode::ZStringView`, which is a type that attempts to be API-compatible with `string_view`, while keeping a guarantee of null termination. Here are some examples of code that breaks and how to fix it:

TODO