# Hooking / Modifying classes

At the centre of every modder's toolkit is **hooking**. If you don't know what that is, [read the handbook](/handbook/chap0.md). If you know what that is and have made mods using gd.h + cocos-headers before, you should know how it works in Geode.

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

The syntax is `class $modify(ModifiedClassName)`, which is selected mostly for aesthetics and not breaking the syntax highlighting. Inside this class, we add the functions we will modify. The example will replace the implementation of `MenuLayer::onMoreGames`, making it spawn a [FLAlertLayer](/tutorials/popup.md) instead of the More Games screen.

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

You will probably encounter an issue with `$modify` in your very first use case: [**how do you add callbacks for buttons**](/tutorials/buttons.md)? By default, `$modify` gives the inherited class a pretty much random name, but if you want to reference the name of your modified class, that's not very ideal as it's not consistent. Luckily, `$modify` can take a second parameter that specifies the name of the class:

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

The syntax for this is `class $modify(MyClassName, ClassToModify)`. 

Creating a `CCMenuItemSpriteExtra` takes a `SEL_MenuHandler`, which is a type alias for `void(cocos2d::CCObject::*)(cocos2d::CCObject*)` (which means a pointer to a member function that has a single parameter of CCObject\*). In order to supply this we need to get access to our function address, which requires us to know the class name. When we don't supply a class name, the macro generates a class which is guaranteed to not create any collision problems. [Read more about menu selectors here](/tutorials/buttons.md)

## Modifying destructors

Mostly because of specific cases of destructors, modifying them is a bit different. You need to create a function named `destructor` instead of a destructor to modify them. Here is an example:

```cpp
class $modify(FLAlertLayer) {
    void destructor() {
        FLAlertLayer::~FLAlertLayer();
        log::info("this alert layer is destructed: {}", this);
    }
};
```

You still need to call the base destructor itself for calling the original.

**This pattern is the same for constructors.**

## Using without macros

If you need to refrain from using the `$modify` macro (clang-format really cries when it sees the macro so), here is how you would do it:

```cpp
struct MyCoolModification : Modify<MyCoolModification, MenuLayer> {
    void onMoreGames(CCObject* target) {
        FLAlertLayer::create(
            "Geode",
            "Hello World from my Custom Mod!",
            "OK"
        )->show(); 
    }
};
```

Yeah we improved the clean usage a lot, and honestly I prefer this over the macro. But who am I to judge?

# Adding members

Geode allows you to add members to your modified classes to extend GD classes nearly seamlessly. [Learn more about fields](/tutorials/fields.md)

# Advanced usage

## Accessing the hooks

Even with this ~~sugary~~ (wow i hate that word) clean syntax hooking, there are still some stuff we can't do without a way of interacting with the hooks, like giving it a priority, manually enabling/disabling even though I'm against it. That's why there is the `onModify` function. This function is called when the hooks are registered to Geode, more specifically the static constructor time of the mod load. Here is how:

```cpp
class $modify(MenuLayer) {
    static void onModify(auto& self) {
        auto res = self.getHook("MenuLayer::init");
        if (!res) {
            log::error("Something went horribly wrong");
            return;
        }
        log::info("I hooked {}!", res.unwrap()->getDisplayName());
    }

    bool init() {
        if (!MenuLayer::init()) return false;
        log::debug("Hi mom I made a modding framework with some friends!")
        return true;
    }
};
```

See the ModifyBase<ModifyDerived> class to see what functions are available (there aren't much to be honest). Also please don't manually enable/disable hooks unless you need to. Oh also did I tell you to never manually enable/disable hooks? Do you still need it? Well, you will need to use `Hook::setAutoEnable` for Geode to not automatically enable your hooks when the mod is enabled, so have fun with that I guess. Just one more thing: please don't manually enable/disable your hooks unless you have to.

## Setting hook priority

You might want to do this because your code may need to run before others ([Geode string id system](/tutorials/nodetree.md) for example). Or you don't call the original so you might want your function to get called last. This is where the priority system is helpful. Here is how you use it:

```cpp
class $modify(GJGarageLayer) {
    static void onModify(auto& self) {
        self.setHookPriorityPost("GJGarageLayer::init", Priority::Early);
    }

    bool init() {
        if (!GJGarageLayer::init()) return false;
        NodeIDs::get()->provide(this);
        return true;
    }
};
```

You can find more details on hook priority at the [Hook Priority tutorial](/tutorials/hookpriority.md).

## Adding delegates

Sometimes you need to add delegates to classes. This is how you would do that:

```cpp
struct $modify(MyDelegateClass) {
    struct Fields : TextInputDelegate {
        MenuLayer* self;
        std::string myString;
        
        void textChanged(CCTextInputNode* node) override {
            // here, this points to m_fields, so you will need that self
            // variable in order to access the MenuLayer itself
            myString = node->getString();
        }
    };
    
    bool init() {
        if (!MenuLayer::init()) return false;
        m_fields->self = this;

        auto input = CCTextInputNode::create(250, 20, "Enter text", "bigFont.fnt");
        input->setDelegate(m_fields.self());
        input->setPosition(100, 100);
        this->addChild(input);

        return true;
    }

    void onMoreGames(CCObject* target) {
        FLAlertLayer::create(
            "Geode",
            "Your text is: " + m_fields->myString,
            "OK"
        )->show(); 
    }
};
```

## Adding new virtual functions

We currently don't support this yet, but here is how you *would* do it anyway (I promise I will add this soon :tm:):

```cpp
class $modify(CopyPlayerObject, PlayerObject) {
    static void onModify(auto& self) {
        self.registerVirtual(&CopyPlayerObject::copyWithZone);
    }

    CCObject* copyWithZone(CCZone* zone) {
        log::info("This function is a meme really");
    }
};
```

## Creating a custom modifier

Not yet implemented

# Common mistakes 

Here are some common mistakes you (and we) may make:

## Accidental recursion

```cpp
class $modify(MenuLayer) {
    void onMoreGames(CCObject* target) {
        log::info("MenuLayer::onMoreGames called woooo");
        this->onMoreGames(target); // !!
    }
};
```

This will result in an infinite loop, because `this->` calls the modified MenuLayer's `onMoreGames` instead of the original's. Name the class you're calling the function from instead (`MenuLayer::onMoreGames`).

## Accidental not recursion

```cpp
class $modify(MenuLayer) {
    bool init() {
        if (!this->init()) return false; // !!

        auto label = CCLabelBMFont::create("Hello world!", "bigFont.fnt");
        label->setPosition(100, 100);
        this->addChild(label);

        return true;
    }
};
```

This will not result in an infinite loop because `MenuLayer::init` is a virtual function, but please don't do `this->function`. It will confuse us all.

## Accidentally not using the correct function

```cpp
class $modify(MyBrokenClass, MenuLayer) {
    void onMoreGames(CCObject* target) {
        log::info("MenuLayer::onMoreGames called woooo");
        MenuLayer::onMoreGames(target);
    }

    bool init() {
        if (!MenuLayer::init()) return false;

        auto buttonSprite = CCSprite::createWithSpriteFrameName("GJ_stopEditorBtn_001.png");
        auto button = CCMenuItemSpriteExtra::create(
            buttonSprite, this,
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

This will result in the modification getting called twice when you press the newly created button, then the original function will be called. (MyBrokenClass::onMoreGames -> handler -> MyBrokenClass::onMoreGames -> handler -> MenuLayer::onMoreGames)

## Not supplying the correct signature

```cpp
class $modify(MenuLayer) {
    bool onMoreGames(CCObject* target) { // !!
        FLAlertLayer::create(
            "Geode",
            "Hello World from my Custom Mod!",´
            "OK"
        )->show(); 
        return true;
    }
};
```

This will result in not modifying the intended function at all, because `onMoreGames` has return type `void` in MenuLayer, not `bool`. I previously tried to implement a check for this in the old version of the modify, but failed miserably. I should honestly try again, but I'm kinda scared that it may increase compile times.
