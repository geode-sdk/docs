# Buttons

One of the most common UI elements in GD is the humble **button**. I'm sure you already know what it is, so let's get straight to the point. Buttons are most commonly created in the form of the `CCMenuItemSpriteExtra` class, which is a GD-specific derivation of the `CCMenuItemSprite` class. You can also use any class that inherits from `CCMenuItem` as a button. You can also technically make your wholly custom button system using the touch system, however that will not be explained in this document.

Every button that inherits `CCMenuItem` **must be a child of a `CCMenu` to work**. This means that if you add a button as a child to a `CCLayer`, you will find that it can't be clicked.

The first argument of `CCMenuItemSpriteExtra` is a `CCNode*` parameter. This is the texture of the button; it can be any `CCNode`, like a label or even a whole layer, but usually most people use a sprite.

If you want to create a button with some text, the most common option is the `ButtonSprite` class. If you want to create something like a circle button (like the 'New' button in GD), you can either use `CCSprite` or [the Geode-specific `CircleButtonSprite` class](#circle-button-sprites).

```cpp
class $modify(MenuLayer) {
    bool init() {
        auto menu = CCMenu::create();

        auto spr = ButtonSprite::create("Hi mom!");

        auto btn = CCMenuItemSpriteExtra::create(
            spr, this, nullptr
        );
        menu->addChild(btn);
    }
};
```

This creates a button with the text `Hi mom!`.

### Callbacks

Button callbacks are called **menu selectors**, and are passed as the third argument to `CCMenuItemSpriteExtra::create`. Menu selectors are non-static class member functions that return `void` and take a single `CCObject*` parameter. For example, this is a menu selector:

```cpp
class MyLayer : public CCLayer {
public:
    void onButton(CCObject* sender);
};
```

It is conventional to have all menu selectors names be in the form of `onSomething`. To pass a menu selector to the button, pass its fully qualified name to the `menu_selector` macro:

```cpp
class MyLayer : public CCLayer {
protected:
    bool init() {
        auto btn = CCMenuItemSpriteExtra::create(
            /* sprite */,
            this,
            menu_selector(MyLayer::onButton)
        );
    }

public:
    void onButton(CCObject* sender) {
        std::cout << "Button clicked!\n";
    }
};
```

Inside the `onButton` function, you have access to the class `this` pointer. The `sender` parameter is a pointer to the button that was clicked. You can cast it back to the `CCMenuItemSpriteExtra` class using `static_cast`:

```cpp
class MyLayer : public CCLayer {
    void onButton(CCObject* sender) {
        std::cout << "Button clicked!\n";
        auto btn = static_cast<CCMenuItemSpriteExtra*>(sender);
        // do something with the button
    }
};
```

> :warning: You can also use `reinterpret_cast` instead of `static_cast`, but using `reinterpret_cast` is generally considered bad practice.

### Example

Here is the popular click counter example in cocos2d:

```cpp
class MyLayer : public CCLayer {
protected:
    // class member that stores how many times 
    // the button has been clicked
    size_t m_clicked = 0;

    bool init() {
        if (!CCLayer::init())
            return false;

        auto menu = CCMenu::create();

        auto btn = CCMenuItemSpriteExtra::create(
            ButtonSprite::create("Click me!"),
            this,
            menu_selector(MyLayer::onClick)
        );
        btn->setPosition(100.f, 100.f);
        menu->addChild(btn);

        this->addChild(menu);

        return true;
    }

    void onClick(CCObject* sender) {
        // increment click count
        m_clicked++;

        auto btn = static_cast<CCMenuItemSpriteExtra*>(sender);

        // getNormalImage returns the sprite of the button
        auto spr = static_cast<ButtonSprite*>(btn->getNormalImage());
        spr->setString(CCString::createWithFormat(
            "Clicked %d times", m_clicked
        )->getCString());
    }
};
```

### Passing more parameters to callbacks

One of the most common problems encountered when using menu selectors is situations where you want to pass more parameters to a function. For example, what if in the click counter example we also wanted to add a decrement button? We could of course just refreace the whole `onClick` function, but that would be quite wasteful. Instead, we can use **tags**.

```cpp
class MyLayer : public CCLayer {
protected:
    // class member that stores how many times 
    // the button has been clicked
    size_t m_clicked = 0;

    bool init() {
        if (!CCLayer::init())
            return false;

        auto menu = CCMenu::create();

        auto btn = CCMenuItemSpriteExtra::create(
            ButtonSprite::create("Click me!"),
            this,
            menu_selector(MyLayer::onClick)
        );
        btn->setPosition(100.f, 100.f);
        btn->setTag(1);
        menu->addChild(btn);

        auto btn2 = CCMenuItemSpriteExtra::create(
            ButtonSprite::create("Decrement"),
            this,
            menu_selector(MyLayer::onClick)
        );
        btn2->setPosition(100.f, 60.f);
        btn2->setTag(-1);
        menu->addChild(btn2);

        this->addChild(menu);

        return true;
    }

    void onClick(CCObject* sender) {
        // increment or decrement click count
        m_clicked += sender->getTag();

        auto btn = static_cast<CCMenuItemSpriteExtra*>(sender);

        // getNormalImage returns the sprite of the button
        auto spr = static_cast<ButtonSprite*>(btn->getNormalImage());
        spr->setString(CCString::createWithFormat(
            "Clicked %d times", m_clicked
        )->getCString());
    }
};
```

If you want to pass something like strings, you should use `setUserObject` instead.

### Inline callbacks

Geode comes with a handy `makeMenuSelector` function that lets you bypass a lot of the boilerplate that comes with creating menu selectors.

```cpp
class MyLayer : public CCLayer {
protected:
    // class member that stores how many times 
    // the button has been clicked
    size_t m_clicked = 0;

    bool init() {
        if (!CCLayer::init())
            return false;

        auto menu = CCMenu::create();

        auto btn = CCMenuItemSpriteExtra::create(
            ButtonSprite::create("Click me!"),
            this,
            makeMenuSelector([this](CCObject*) {
                // increment click count
                m_clicked++;

                auto btn = static_cast<CCMenuItemSpriteExtra*>(sender);

                // getNormalImage returns the sprite of the button
                auto spr = static_cast<ButtonSprite*>(btn->getNormalImage());
                spr->setString(CCString::createWithFormat(
                    "Clicked %d times", m_clicked
                )->getCString());
            })
        );
        btn->setPosition(100.f, 100.f);
        menu->addChild(btn);

        this->addChild(menu);

        return true;
    }
};
```

`makeMenuSelector` does come with some caveats; every instance of the same menu selector shares the same captures. This means that if you assign inline menu selectors in a loop and capture the current index, each button created in the loop has the same value despite what you might expect.

### Circle button sprites

Geode comes with a concept known as **based button sprites**, which are button sprites that come with common GD button backgrounds and let you add your own sprite on top. These are useful for texture packs, as texture packers can just style the bases and don't have to individually make every mod fit their pack's style.

```cpp
#include <Geode/ui/BasedButtonSprite.hpp>

auto spr = CircleButtonSprite::createWithSpriteFrameName("top-sprite.png"_spr);
```
