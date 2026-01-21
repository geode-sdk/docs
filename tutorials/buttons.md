# Buttons

One of the most common UI elements in GD is the humble **button**. I'm sure you already know what it is, so let's get straight to the point. Buttons are most commonly created in the form of the `CCMenuItemSpriteExtra` class, which is a GD-specific derivation of the `CCMenuItemSprite` class. However, using this directly takes a lot of steps, so Geode has it's own utility to easily create a `CCMenuItemSpriteExtra`, `CCMenuItemExt::createSpriteExtra`. This utility also contains other classes such as `CCMenuItemSprite`. See [this page](https://docs.geode-sdk.org/classes/geode/cocos/CCMenuItemExt) for all of the button classes available. You can also use any class that inherits from `CCMenuItem` as a button.

Every button that inherits `CCMenuItem` **must be a child of a `CCMenu` to work**. This means that if you add a button as a child to a `CCLayer`, you will find that it can't be clicked.

The first argument of `CCMenuItemExt::createSpriteExtra` is a `CCNode*` parameter. This is the texture of the button; it can be any `CCNode`, like a label or even a whole layer, but usually most people use a sprite.

If you want to create a button with some text, the most common option is the `ButtonSprite` class. If you want to create something like a circle button (like the 'New' button in GD), you can either use `CCSprite` or [the Geode-specific `CircleButtonSprite` class](#circle-button-sprites).

```cpp
bool MyLayer::init() {
    // ...

    auto spr = ButtonSprite::create("Hi mom!");

    auto btn = CCMenuItemExt::createSpriteExtra(
        spr, 
        nullptr
    );

    // some CCMenu*
    menu->addChild(btn);

    // ...
}
```

This creates a button with the text `Hi mom!`.

## Callbacks
Button callbacks are passed as the second argument to `CCMenuItemExt::createSpriteExtra` in a lambda. Here's an example on how you can make a callback with `CCMenuItemExt::createSpriteExtra`:

```cpp
class MyLayer : public CCLayer {
protected:
    bool init() {
        // ...

        auto btn = CCMenuItemExt::createSpriteExtra(
            /* sprite */,
            [this](CCMenuItemSpriteExtra* btn) {
                log::info("Button clicked!");
            }
        );

        // ...
    }
};
```

## Example

Here is the popular click counter example in cocos2d:

```cpp
class MyLayer : public CCLayer {
protected:
    // Class member that stores how many times 
    // the button has been clicked
    size_t m_clicked = 0;

    bool init() {
        if (!CCLayer::init()) return false;

        auto menu = CCMenu::create();

        auto btn = CCMenuItemExt::createSpriteExtra(
            ButtonSprite::create("Click me!"),
            [this](CCMenuItemSpriteExtra* btn) { // This is where we add the callback!
                m_clicked++;
                
                // getNormalImage returns the sprite of the button
                auto spr = static_cast<ButtonSprite*>(btn->getNormalImage());
                spr->setString(fmt::format("Clicked {} times", m_clicked).c_str());
            }
        );

        btn->setPosition(100.f, 100.f);
        menu->addChild(btn);

        this->addChild(menu);

        return true;
    }
};
```

## Passing more parameters to callbacks
One common issue when working with button callbacks is handling situations where you want to pass additional parameters to a function. For example, what if in the click counter example we also wanted to add a decrement button? We could write a separate callback for each button, but that would be unnecessary duplication. Instead, we can simply pass parameters **directly through lambda captures**.

```cpp
class MyLayer : public CCLayer {
protected:
    // Class member that stores how many times
    // the button has been clicked
    size_t m_clicked = 0;

    // Label that displays the number of clicks
    CCLabelBMFont* m_label = nullptr;

    void updateCounter(int delta) {
        m_clicked += delta;
        m_label->setString(fmt::format("Clicked: {}", m_clicked).c_str());
    }

    bool init() {
        if (!CCLayer::init()) return false;

        m_label = CCLabelBMFont::create("Clicked: 0", "bigFont.fnt");
        m_label->setPosition(100.f, 150.f);
        this->addChild(m_label);

        auto menu = CCMenu::create();

        // Increment button
        auto btn = CCMenuItemExt::createSpriteExtra(
            ButtonSprite::create("+1"),
            [this](auto) { this->updateCounter(1); }
        );
        btn->setPosition(100.f, 100.f);
        menu->addChild(btn);

        // Decrement button
        auto btn2 = CCMenuItemExt::createSpriteExtra(
            ButtonSprite::create("-1"),
            [this](auto) { this->updateCounter(-1); }
        );
        btn2->setPosition(100.f, 60.f);
        menu->addChild(btn2);

        this->addChild(menu);

        return true;
    }
};
```

# Based button sprites

Geode comes with a concept known as **based button sprites**, which are button sprites that come with common GD button backgrounds and let you add your own sprite on top. These are useful for texture packs, as texture packers can just style the bases and don't have to individually make every mod fit their pack's style.

> :information_source: This section only explains how to create based button sprites from a sprite frame name. For other creation methods, check the documentation of each button sprite class.

## Circle button sprites
By using [CircleButtonSprite](https://docs.geode-sdk.org/classes/geode/CircleButtonSprite/), you can create button sprites similar to the buttons at the bottom of the main menu.

### Example
Here's an example on how you can create a circle button sprite programmatically:

```cpp
#include <Geode/ui/BasedButtonSprite.hpp>

using namespace geode::prelude;

// ...

auto spr = CircleButtonSprite::createWithSpriteFrameName(
    "sprite.png"_spr, // The sprite that will be added on top of the button background. _spr adds the mod prefix at the start and is required for using a custom sprite. _spr is not required when using GD's sprites.
    1.0f, // The scale of the sprite (float)
    CircleBaseColor::Green, // Available options: Blue, Cyan, DarkAqua, DarkPurple, Gray, Green, Pink
    CircleBaseSize::Medium // Available options: Big, BigAlt, Large, Medium, MediumAlt, Small, SmallAlt, Tiny
);
```
## Category button sprites
By using [CategoryButtonSprite](https://docs.geode-sdk.org/classes/geode/CategoryButtonSprite/), you can create button sprites for category buttons, i.e. the big buttons in the create tab.

### Example
Here's an example on how you can create a category button sprite programmatically:

```cpp
#include <Geode/ui/BasedButtonSprite.hpp>

using namespace geode::prelude;

auto spr = CategoryButtonSprite::createWithSpriteFrameName(
    "sprite.png"_spr, // The sprite that will be added on top of the button background. _spr adds the mod prefix at the start and is required for using a custom sprite. _spr is not required when using GD's sprites.
    1.0f, // The scale of the sprite (float)
    CategoryBaseColor::Green,
    CategoryBaseSize::Big
);
```

## Account button sprites
By using [AccountButtonSprite](https://docs.geode-sdk.org/classes/geode/AccountButtonSprite/), you can create button sprites with a cross base, like the buttons in the main menu.

### Example
Here's an example on how you can create a category button sprite programmatically:

```cpp
#include <Geode/ui/BasedButtonSprite.hpp>

using namespace geode::prelude;

auto spr = AccountButtonSprite::createWithSpriteFrameName(
    "sprite.png"_spr, // The sprite that will be added on top of the button background. _spr adds the mod prefix at the start and is required for using a custom sprite. _spr is not required when using GD's sprites.
    1.0f, // The scale of the sprite (float)
    AccountBaseColor::Blue, // Available options: Blue, Gray, Purple
    AccountBaseSize::Normal
);
```
## Editor button sprites
By using [EditorButtonSprite](https://docs.geode-sdk.org/classes/geode/AccountButtonSprite/), you can create button sprites with the same base as the right-side action buttons in the level editor.

### Example
Here's an example on how you can create a editor button sprite programmatically:

```cpp
#include <Geode/ui/BasedButtonSprite.hpp>

using namespace geode::prelude;

auto spr = EditorButtonSprite::createWithSpriteFrameName(
    "sprite.png"_spr, // The sprite that will be added on top of the button background. _spr adds the mod prefix at the start and is required for using a custom sprite. _spr is not required when using GD's sprites.
    1.0f, // The scale of the sprite (float)
    EditorBaseColor::LightBlue, // Available options: LightBlue, Green, Orange, DarkGray, Gray, Pink, Teal, Aqua, Cyan, Magenta, DimGreen, BrightGreen, Salmon
    EditorBaseSize::Normal
);