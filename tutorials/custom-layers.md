# Custom Layers

At some point a modder may want to create a custom layer for their mod, for one reason or another.


Creating a new scene is easy. You must create a new class extending `cocos2d:::CCLayer`, and add additional UI (such as buttons, popups, or even a level) by overriding the the `*::init` function.

```cpp
class MyVeryOriginalLayer : public CCLayer {
protected:
    bool init() override {
        log::info("Hi from my very original layer");
        return true;
    }

public:
    static MyVeryOriginalLayer* create() {
        auto ret = new MyVeryOriginalLayer;
        if (ret->init()) {
            ret->autorelease();
            return ret;
        }
        CC_SAFE_DELETE(ret);
        return nullptr;
    }
}
```

This code will create a new layer called MyVeryOriginalLayer. When we start up the game, we will not see the layer as we do transition to it. Calling `MyVeryOriginalLayer::create()` will not transition to the layer, as it only creates it, not replaces the current layer. To be able to transition to the layer, we can use `switchToScene(layer)`. We can create a static function to easily create and switch to the scene:

```cpp
static MyVeryOriginalLayer* scene() {
    auto layer = MyVeryOriginalLayer::create();
    switchToScene(layer);
    return layer;
}
```

## UI

Running this code and calling `MyVeryOriginalLayer::scene()` will transition the current scene to MyVeryOriginalLayer, however it is currently very barren. Geode provides utility functions to create background and side art.

```
#include <Geode/ui/General.hpp>

bool init() override {
    log::info("Hi from my very original layer");

    // Create a new menu. UI elemnts should be added to here!
    menu = CCMenu::create();

    // Add side art to the layer
    addSideArt(this, SideArt::All, SideArtStyle::Layer);

    // And a background to the layer
    auto background = createLayerBG();
    background->setID("background");
    this->addChild(background);	
    return true;
}
```

The rest of the buttons and UI can be created the same way you would in a hook.

> :warning: UI elements should **always** be added to a CCMenu object!


## Back button

Now the biggest problem with this layer is that you are stuck in it. To go back you would want to create a button whoes callback transitions to another scene. 

```cpp

bool init() override {
    // Create a back button with the back button sprites
    auto backBtn = CCMenuItemSpriteExtra::create(
        CCSprite::createWithSpriteFrameName("GJ_arrow_01_001.png"),
        this,
        menu_selector(MyVeryOriginalLayer::onBack)
    );


    // Add a back button the the top-left corner of the screen with a small offset.
    menu->addChildAtPosition(backBtn, Anchor::TopLeft, {25, -25});
    this->addChild(menu);
    return true;
}

// This function is called when the escape key is pressed!
void keyBackClicked() override {
    this->onBack(nullptr);
}

void onBack(CCObject* sender) {
    CCDirector::get()->replaceScene(CCTransitionFade::create(0.5, MenuLayer::scene()));
}

```

> :information_source: You can also create a back button with `GameToolbox::addBackButton`

> :warning: Pressing escape will **not transition the layer** back unless you override `keyBackClicked()`!



## Example

This code will transition to the MyVeryOriginalLayer when clicking the on more games button.

```cpp
#include <Geode/Geode.hpp>
#include <Geode/ui/General.hpp>
#include <Geode/modify/MenuLayer.hpp>
using namespace geode::prelude;

class MyVeryOriginalLayer : public CCLayer {
protected:
    bool init() override {
        log::info("Hi from my very original layer");
        
        // Create a new menu. UI elemnts should be added to here!
        menu = CCMenu::create();
        
        // Add side art to the layer
        addSideArt(this, SideArt::All, SideArtStyle::Layer);
        
        // And a background to the layer
        auto background = createLayerBG();
        background->setID("background");
        this->addChild(background);	
    return true;
    }

    // This function is called when the escape key is pressed!
    void keyBackClicked() override {
        this->onBack(nullptr);
    }

public:
    static MyVeryOriginalLayer* create() {
        auto ret = new MyVeryOriginalLayer;
        if (ret->init()) {
            ret->autorelease();
            return ret;
        }
        CC_SAFE_DELETE(ret);
        return nullptr;
    }

    static MyVeryOriginalLayer* MyVeryOriginalLayer::scene() {
        auto layer = MyVeryOriginalLayer::create();
        switchToScene(layer);
        return layer;
    }


    void onBack(CCObject* sender) {
        CCDirector::get()->replaceScene(CCTransitionFade::create(0.5, MenuLayer::scene()));
    }

}

class $modify(MenuLayer) {
    void onMoreGames(CCObject* target) {
        MyVeryOriginalLayer::scene();
    }
};
```
