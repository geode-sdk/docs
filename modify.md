---
layout: default
title: Modifying classes
parent: Usage
nav_order: 3
description: "Modifying classes"
---

# Modifying classes

Haven't found what you need inside the api? Do you want to modify the behaviour of a Geometry Dash class because it is bad? Do you want implement new features? Look no further! Geode has the tool for you!

## Modifying a function

Modifying classes is done using the macro `$modify`. If a function inside GD is implemented (or a custom modifier exists, which will be explained a lot later), you can use this macro to alter its behaviour. 

For example, to change the behaviour of the "More Games" button in GD, all you have to do is this:

```cpp
class $modify(MenuLayer) {
    void onMoreGames(CCObject* target) {
        FLAlertLayer::create(
            "Geode",
            "Hello World from my Custom Mod!",
            "OK"
        )->show(); 
    }
};
```

Wait wait wait, so what does this exactly do?

The function we are modifying is `MenuLayer::onMoreGames`, which is called when you press the "More Games" button. This function takes a single parameter, a `cocos2d::CCObject*`, which is the target object the function is called on. For our use case, we don't need this parameter. 

The syntax is `class $modify(ModifiedClassName)`, which is selected mostly for aesthetics and not breaking the syntax highlighting. Inside this class, we add the functions we will modify. The example will replace the implementation of `MenuLayer::onMoreGames`, making it spawn a FLAlertLayer instead of the More Games screen.

## Using the original function

Sometimes you will want to not replace, but extend on a function. This is why you may (and mostly do) need to access the original function. This is done in the same way C++ handles base functions. In order to call a function from a base class, you specify the class and function name together. Here is an example:

```cpp
class $modify(MenuLayer) {
    bool init() {
        if (!MenuLayer::init()) return false;

        auto label = CCLabelBMFont::create("Hello world!", "bigFont.fnt");
        label->setPosition(100, 100);
        this->addChild(label);

        return true;
    }
};
```

(or you could be questionable and do `this->MenuLayer::init()` but who am I to judge)

This pattern is quite common inside cocos2d, and therefore in GD too. Here, we modify the `MenuLayer::init` so that it will create our label inside the layer after running the original function. If `MenuLayer::init` returns false something has gone horribly wrong and GD will probably crash but it is important to stick to the idioms for code consistency. 

## Giving a class name

Maybe you want to name your classes because of needing to categorize your modifications, maybe you need to access a callback function, and this is why you can name your classes with the `$modify` macro. Here is an example:

```cpp
class $modify(MyAwesomeModification, MenuLayer) {
    void onMyButton(CCObject* target) {
        FLAlertLayer::create(
            "Geode",
            "Hello World from my Custom Mod!",
            "OK"
        )->show(); 
    }

    bool init() {
        if (!MenuLayer::init()) return false;

        auto buttonSprite = CCSprite::createWithSpriteFrameName("GJ_stopEditorBtn_001.png");
        auto button = CCMenuItemSpriteExtra::create(
            buttonSprite, nullptr, this,
            menu_selector(MyAwesomeModification::onMyButton)
        );

        auto menu = CCMenu::create();
        menu->addChild(button);
        menu->setPosition(150, 100);
        this->addChild(menu);

        return true;
    }
};
```

The syntax for this is `class $modify(MyClassName, ModifiedClassName)`. It looks cool.

Creating a `CCMenuItemSpriteExtra` takes a `SEL_MenuHandler`, which is a type alias for `void(cocos2d::CCObject::*)(cocos2d::CCObject)` (which means a pointer to a member function that has a single parameter of CCObject\*). In order to supply this we need to get access to our function address, which requires us to know the class name. When we don't supply a class name, the macro generates a class which is guaranteed to not create any collision problems. 

## Modifying destructors

Mostly because of specific cases of destructors, modifying them is a bit different. You need to create a function named `destructor` instead of a destructor to modify them. Here is an example:

```cpp
class $modify(FLAlertLayer) {
    void destructor() {
        FLAlertLayer::~FLAlertLayer();
        Log::get() << "this alert layer is destructed: " << this;
    }
};
```

You still need to call the base destructor itself for calling the original.

# Adding fields

Well modifying the functions are good and all but, I need to add some members to my beatiful class `MenuLayer` that I love to massacre. How do I do that? 

I present to you, fields!

## Adding a field

There is a class inside the `geode::modifier` namespace called `field`. This class allows you to introduce members into the classes you modify. Here is an example:

```cpp
class $modify(PlayerObject) {
    field<int> totalJumps;

    void pushButton(PlayerButton button) {
        Log::get() << "The player has jumped " << this->*totalJumps << " times !";
        this->*totalJumps += 1;
        PlayerObject::pushButton(button);
    }
};
```

This provides you with an `int` variable for every `PlayerObject` that gets created. It is initialized to the default `int` value, which is 0. Every field gets initialized with its default value. 

However, using the field is a bit, unusual let's say. The syntax for accessing the field is `this->*myField`. The operator `->*` is needed. This is also the reason why something like `++this->*totalJumps` wouldn't work, and you would need to put the member inside parenthesis since the `++` operator [has higher precedence](https://en.cppreference.com/w/cpp/language/operator_precedence) than `->*`. That is why I like `this->*totalJumps += 1` more, since it doesn't need the ugly parenthesis.

## Setting a default value

To set a default value, you can do it by this:

```cpp
class $modify(PlayerObject) {
    field<int> remainingJumps = 50;

    void pushButton(PlayerButton button) {
        Log::get() << "The player has " << this->*remainingJumps << " jumps remaining!";
        this->*remainingJumps -= 1;
        PlayerObject::pushButton(button);
    }
};
```

(oh wow this took me so much time to get it working)

# Advanced usage

## Modifier specific functions

In some cases, you may need to specify some properties into a modification, or you may need to run some functions when a modification is applied. Okay I'm being too vague here because the use case of these is absolutely zero without explaining the other parts. So bare with me.

In order to do this, you can create a static function named `onApplyModification` (TBD), which will run when the modification is done. Here is an example:

```cpp
class $modify(MenuLayer) {
    template <class Modify> 
    void onApplyModification(Modify modify) {
        Log::get() << "My label will now exist when you open the menu layer :)";
    }

    bool init() {
        if (!MenuLayer::init()) return false;

        auto label = CCLabelBMFont::create("Hello world!", "bigFont.fnt");
        label->setPosition(100, 100);
        this->addChild(label);

        return true;
    }
};
```

## Using without macros

Creating a modification uses the `$modify` macro but if you are a cool kid you can not use it. Here is how:

```cpp
class MyCoolModification : public MenuLayer {
public:
    void onMoreGames(CCObject* target) {
        FLAlertLayer::create(
            "Geode",
            "Hello World from my Custom Mod!",
            "OK"
        )->show(); 
    }
};
Modify<MyCoolModification, MenuLayer> someDummyName;
```

It wasn't that bad was it?

## Adding delegates

We currently don't have a shortcut way of implementing this, sorry. But if you need it for some reason here is how you would do it:

```cpp
class $modify(MyComplicatedClass, MenuLayer) {
    struct TextInput : TextInputDelegate {
        TextInput(MyComplicatedClass* ptr) : self(ptr) {}
        MyComplicatedClass* self;
        void textChanged(CCTextInputNode* node) {
            self->*myString = node->getString();
        }
    }

    field<TextInput> input;
    field<std::string> myString;
    
    bool init() {
        if (!MenuLayer::init()) return false;
        this->*input = this;

        auto input = CCTextInputNode::create(250, 20, "Enter text", "bigFont.fnt");
        input->setDelegate(&(this->*input));
        input->setPosition(100, 100);
        this->addChild(input);

        return true;
    }

    void onMoreGames(CCObject* target) {
        FLAlertLayer::create(
            "Geode",
            "Your text is: " this->*myString,
            "OK"
        )->show(); 
    }
};
```

## Adding new virtual functions

We currently don't support this yet, but here is how you would do it anyway:

```cpp
class $modify(CopyPlayerObject, PlayerObject) {
    template <class Modify> 
    void onApplyModification(Modify modify) {
        modify.registerVirtualFunction(&CopyPlayerObject::copyWithZone);
    }

    CCObject* copyWithZone(CCZone* zone) {
        Log::get() << "This function is a meme really";
        copyWithZone(zone);
    }
};
```

## Creating a custom modifier

Not yet implemented

# Common mistakes 

Here are some common mistakes you (and us) may make:

## Accidental recursion

```cpp
class $modify(MenuLayer) {
    bool init() {
        if (!this->init()) return false;

        auto label = CCLabelBMFont::create("Hello world!", "bigFont.fnt");
        label->setPosition(100, 100);
        this->addChild(label);

        return true;
    }
};
```

This will result in an infinite loop.

## Accidental not using the correct function

```cpp
class $modify(MyBrokenClass, MenuLayer) {
    void onMoreGames(CCObject* target) {
        Log::get() << "MenuLayer::onMoreGames called woooo";
        MenuLayer::onMoreGames(target);
    }

    bool init() {
        if (!MenuLayer::init()) return false;

        auto buttonSprite = CCSprite::createWithSpriteFrameName("GJ_stopEditorBtn_001.png");
        auto button = CCMenuItemSpriteExtra::create(
            buttonSprite, nullptr, this,
            menu_selector(MyBrokenClass::onMoreGames)
        );

        auto menu = CCMenu::create();
        menu->addChild(button);
        menu->setPosition(150, 100);
        this->addChild(menu);

        return true;
    }
};
```

This will result in the modification getting called twice when you press the newly created button, then the original function will be called.

## Accidental not supplying the correct signature

```cpp
class $modify(MenuLayer) {
    bool onMoreGames(CCObject* target) {
        FLAlertLayer::create(
            "Geode",
            "Hello World from my Custom Mod!",´
            "OK"
        )->show(); 
        return true;
    }
};
```

This will result in not modifying the intended function at all.
