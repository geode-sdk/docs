# Buttons
One of the most common UI elements in GD is the humble **button**. I'm sure you already know what it is, so let's get straight to the point. Buttons are most commonly created using the `CCMenuItemSpriteExtra` class, which is a GD-specific derivation of `CCMenuItemSprite`.
Creating callbacks with `CCMenuItemSpriteExtra` normally requires writing separate callback functions, which can make UI code unnecessarily verbose. Geode provides `CCMenuItemExt` to solve this. `CCMenuItemExt` is a wrapper around `CCMenuItem` that adds convenient factory methods for creating existing `CCMenuItem` classes with lambda callbacks. For example, `createSpriteExtra` creates a normal `CCMenuItemSpriteExtra`, but lets you define its callback inline instead of writing a separate function. See the `CCMenuItemExt` documentation by clicking [here](https://docs.geode-sdk.org/classes/geode/cocos/CCMenuItemExt) for the full list of available factory methods.

You can also use any class that inherits from `CCMenuItem` as a button.

Every button that inherits `CCMenuItem` (which both `CCMenuItemSpriteExtra` and the `CCMenuItemExt` wrapper do) **must be a child of a `CCMenu` to work**. This means that if you add a button as a child to a `CCLayer`, you will find that it can't be clicked. If you don't want to make a `CCMenu`, see the [**menuless buttons**](#menuless-buttons) section, which explains Geode's `Button` class.
For `CCMenuItem`-based buttons, **scale the sprite, not the button**. Scaling the button itself **will cause its scale to reset when it is clicked**. Geode's `Button` class does not have this limitation, so it can be scaled directly.

The first parameter of `CCMenuItemExt::createSpriteExtra` is a `CCNode*`. This is the display node of the button; it can be any `CCNode`, like a label or even a whole layer, but usually most people use a sprite.

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
Button callbacks are passed as the second parameter to `CCMenuItemExt::createSpriteExtra` as a lambda. If you want to access members of your layer inside the callback, capture `this`. Otherwise, you don't need to capture anything. Here's an example of creating a callback with `CCMenuItemExt::createSpriteExtra`:

```cpp
class MyLayer : public CCLayer {
protected:
    bool init() {
        // ...

        auto btn = CCMenuItemExt::createSpriteExtra(
            /* sprite */,
            [](CCMenuItemSpriteExtra*) {
                log::info("Button clicked!");
            }
        );

        // ...
    }
};
```

## Example

Here is the popular click counter example in cocos2d-x using `CCMenuItemExt`:

```cpp
class MyLayer : public CCLayer {
protected:
    // Class member that stores how many times
    // the button has been clicked
    int m_clicked = 0;

    bool init() {
        if (!CCLayer::init()) return false;

        auto menu = CCMenu::create();

        auto btn = CCMenuItemExt::createSpriteExtra(
            ButtonSprite::create("Click me!"),
            [this](CCMenuItemSpriteExtra* sender) { // This is where we add the callback!
                // Increment the counter.
                m_clicked++;

                // getNormalImage returns the display node of the button.
                // Make sure to cast it to the type used by the button.
                // Only use `static_cast` if you're the one who owns the button, else use `typeinfo_cast`.
                auto spr = static_cast<ButtonSprite*>(sender->getNormalImage());
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
    int m_clicked = 0;

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
        addChild(m_label);

        auto menu = CCMenu::create();

        // Increment button
        auto incrementBtn = CCMenuItemExt::createSpriteExtra(
            ButtonSprite::create("+1"),
            [this](CCMenuItemSpriteExtra* sender) { updateCounter(1); }
        );
        incrementBtn->setPosition(100.f, 100.f);
        menu->addChild(incrementBtn);

        // Decrement button
        auto decrementBtn = CCMenuItemExt::createSpriteExtra(
            ButtonSprite::create("-1"),
            [this](CCMenuItemSpriteExtra* sender) { updateCounter(-1); }
        );
        decrementBtn->setPosition(100.f, 60.f);
        menu->addChild(decrementBtn);

        addChild(menu);

        return true;
    }
};
```

## Creating toggles
`CCMenuItemExt` comes with `CCMenuItemToggler` wrappers for creating toggles. One such example is `createTogglerWithStandardSprites`, which automatically places the checkmark sprites for you. Example:

```cpp
bool MyLayer::init() {
    if (!CCLayer::init()) return false;

    auto menu = CCMenu::create();
    addChild(menu);

    auto toggleBtn = CCMenuItemExt::createTogglerWithStandardSprites(
        1.f, // scale
        [](CCMenuItemToggler* sender) {
            if (!sender->isToggled()) { // Due to a quirk, you have to invert the condition.
                log::info("Button toggled on!");
            } else {
                log::info("Button toggled off!");
            }
        }
    );
    toggleBtn->setPosition(100.f, 100.f);
    menu->addChild(toggleBtn);

    return true;
}
```

# Menuless buttons
If you don't want to create a `CCMenu` to support callbacks on your button, Geode has a `Button` class for that. This class allows you to create buttons with callbacks without needing a `CCMenu`, since unlike the previous classes, this class does not inherit `CCMenuItem`. See [this page](https://docs.geode-sdk.org/classes/geode/Button/) for all of the create methods available.

```cpp
bool MyLayer::init() {
    // ...

    // Use `createWithNode` if you want to create the button with a node (`CCNode`).
    // `ButtonSprite` here is a node.
    // Other methods take the sprite name as a string.
    auto btn = Button::createWithNode(
        ButtonSprite::create("Hi mom!"),
        [](Button*) { 
            // do nothing
        }
    );

    this->addChild(btn); // Add the button directly to the layer! No need to add it to a menu, and the callback will work! :3
}
```

## Callbacks
Button callbacks are usually passed as the second parameter to the `Button` class (the position of the parameter depends on the create method) in a lambda. If you want to access members of your layer inside the callback, capture `this`. Here's an example on how you can make a callback with `Button::createWithNode`:

```cpp
class MyLayer : public CCLayer {
protected:
    bool init() {
        // ...

        auto btn = Button::createWithNode(
            /* sprite */,
            [](Button*) {
                log::info("Button clicked!");
            }
        );

        // ...
    }
};
```

## Example

Here is the popular click counter example in cocos2d-x using `Button`:

```cpp 
class MyLayer : public CCLayer {
protected:
    // Class member that stores how many times
    // the button has been clicked
    int m_clicked = 0;

    bool init() {
        if (!CCLayer::init()) return false;

        auto btn = Button::createWithNode(
            ButtonSprite::create("Click me!"),
            [this](Button* sender) { // This is where we add the callback!
                // Increment the counter.
                m_clicked++;

                // The equivalent of `getNormalImage` in `geode::Button` is `getDisplayNode`.
                // Works exactly like `getNormalImage`, you cast it with the type used by the button.
                // Only use `static_cast` if you're the one who owns the button, else use `typeinfo_cast`.
                auto spr = static_cast<ButtonSprite*>(sender->getDisplayNode());
                spr->setString(fmt::format("Clicked {} times", m_clicked).c_str());
            }
        );

        btn->setPosition(100.f, 100.f);
        addChild(btn);

        return true;
    }
};
```

## Passing more parameters to callbacks
Just like in the earlier `CCMenuItemExt` example, you may run into cases where a `Button` callback needs extra information beyond the sender itself. For instance, suppose we want to extend the click counter with a decrement button as well; writing a separate function for each button would be redundant, so instead we rely on **lambda captures** to pass in the needed parameters.

```cpp
class MyLayer : public CCLayer {
protected:
    // Class member that stores how many times
    // the button has been clicked
    int m_clicked = 0;

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

        // Increment button
        auto incrementBtn = Button::createWithNode(
            ButtonSprite::create("+1"),
            [this](Button*) { this->updateCounter(1); }
        );
        incrementBtn->setPosition(100.f, 100.f);
        this->addChild(incrementBtn);

        // Decrement button
        auto decrementBtn = Button::createWithNode(
            ButtonSprite::create("-1"),
            [this](Button*) { this->updateCounter(-1); }
        );
        decrementBtn->setPosition(100.f, 60.f);
        this->addChild(decrementBtn);

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
Here's an example on how you can create an account button sprite programmatically:

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
By using [EditorButtonSprite](https://docs.geode-sdk.org/classes/geode/EditorButtonSprite/), you can create button sprites with the same base as the right-side action buttons in the level editor.

### Example
Here's an example on how you can create an editor button sprite programmatically:

```cpp
#include <Geode/ui/BasedButtonSprite.hpp>

using namespace geode::prelude;

auto spr = EditorButtonSprite::createWithSpriteFrameName(
    "sprite.png"_spr, // The sprite that will be added on top of the button background. _spr adds the mod prefix at the start and is required for using a custom sprite. _spr is not required when using GD's sprites.
    1.0f, // The scale of the sprite (float)
    EditorBaseColor::LightBlue, // Available options: LightBlue, Green, Orange, DarkGray, Gray, Pink, Teal, Aqua, Cyan, Magenta, DimGreen, BrightGreen, Salmon
    EditorBaseSize::Normal
);
```
