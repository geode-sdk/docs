# Creating Popups

Whenever you're designing an UI, you should always make it clear to the user whether an action is pending, if something has gone wrong and if something is succesful. For this purpose, you could always design your own dedicated solution for showing status, but in many cases that's a good idea, but in many other cases you should just use a common, simple solution: **popups**. They are ubiqutous in GD and in mods, as they are a very simple and quick way of showing the user information.

The standard popup class in GD is `FLAlertLayer`. This class comes with a shaded background, title, text, buttons, and all the other stuff you expect. Using this class is pretty simple too:
```cpp
FLAlertLayer::create(
    "Title",    // title
    "Hi mom!",  // content
    "OK"        // button
)->show();
```
This is the way GD does simple popups and also **the recomended way in Geode** to do simple info popups. If you just need to tell the user some info like that an action was succesful, this is the best way.

### Questions

However, using `FLAlertLayer` directly is **not always ideal**. Say that you want to have multiple buttons in the popup, and do something based on which one was clicked; for example, a confirmation popup for deleting a level. For this case, GD uses the `FLAlertLayerProtocol` member in `FLAlertLayer::create`. However, **this is not recommended**, as doing this comes with a lot of extra boilerplate. To use this, you need to inherit from `FLAlertLayerProtocol` in your class, then override `FLAlert_Clicked`, then pass the layer as the first argument to `FLAlertLayer::create` and in `FLAlert_Clicked` handle the action. And then if you want to introduce multiple popups in the same layer, you will need to keep track of tags and which popup is which and it all becomes a huge mess of unworkable code.

Luckily, **Geode provides a much better alternative**: the `createQuickPopup` function. It takes similar parameters to `FLAlertLayer::create`, but also takes an `std::function` callback for when the popup is clicked. Using it looks like this:
```cpp
geode::createQuickPopup(
    "Title",            // title
    "Say hi to mom?",   // content
    "Nah", "Yeah",      // buttons
    [](auto, bool btn2) {
        if (btn2) {
            // say hi to mom
        }
    }
);
```
This makes working with simple confirmation popups much simpler, as you don't have to bring in any extra inheritance or boilerplate. Just call a function, and pass a callback function for the result. Geode mods are **highly encouraged** to use this over `FLAlertLayerProtocol`. However, as this is not an interoperability concern, you are free to do whatever fits your needs.

### Complex popups

However, what if you want to make a popup that's more complex? For example, something with a different background and lots of buttons and other controls, like the move trigger popup? For this, the **unrecommended** GD way is to inherit from `FLAlertLayer` and override `init` with your own code. However, this is not ideal, as most complex popups share a lot of similar base functionality like a close button and a standard `GJ_squareXX` background. For this reason, Geode provides a convenience class `Popup<...>` for creating your own complex popups.
```cpp
// specify parameters for the setup function in the Popup<...> template
class MyPopup : public geode::Popup<std::string const&> {
protected:
    bool setup(std::string const& value) override {
        auto winSize = CCDirector::sharedDirector()->getWinSize();

        // convenience function provided by Popup 
        // for adding/setting a title to the popup
        this->setTitle("Hi mom!");

        auto label = CCLabelBMFont::create(value.c_str(), "bigFont.fnt");
        label->setPosition(winSize / 2);
        this->addChild(label);

        return true;
    }

public:
    static MyPopup* create(std::string const& text) {
        auto ret = new MyPopup();
        if (ret && ret->init(240.f, 160.f, text)) {
            ret->autorelease();
            return ret;
        }
        CC_SAFE_DELETE(ret);
        return nullptr;
    }
};
```

`Popup` contains one pure virtual member function: `setup`, whose parameters are the class template arguments. This function is called at the end of `Popup::init`, which initializes all the relevant `FLAlertLayer` members and adds a background and a close button. This abstracts away a lot of boilerplate and makes creating complex popups simple. **This is the recommended way to create complex popups in Geode**, although as with question popups this is not an interoperability concern and as such you can do what fits your case.

### Colored text

`FLAlertLayer` supports colored text in the content field by default. You can add colors with **color tags**, for example `<cy>Hi mom!</c>` will produce yellow text. The built-in color tags in GD are:

| Tag | Color                               |
|-----|-------------------------------------|
| cb  | <span color="#4a52e1">Blue</span>   |
| cg  | <span color="#40e348">Green</span>  |
| cl  | <span color="#60abef">Aqua</span>   |
| cj  | <span color="#32c8ff">Cyan</span>   |
| cy  | <span color="#ffff00">Yellow</span> |
| co  | <span color="#ffa54b">Orange</span> |
| cr  | <span color="#ff5a5a">Red</span>    |
| cp  | <span color="#ff00ff">Pink</span>   |

Note that the closing tag **must be `</c>` only without the color specified again**. Doing otherwise will result in undesired behaviour (a crash).

### Disabling the popup animation

You can disable the popup's enter animation by setting the `m_noElasticity` member to `true`.

### Popup not showing up

Sometimes you will want to create a popup and show it in a layer's `init` function. However, if you do something like this:
```cpp
class $modify(MenuLayer) {
    bool init() {
        if (!MenuLayer::init())
            return false;
        
        FLAlertLayer::create(
            "Title",
            "Hi mom!",
            "OK"
        )->show();

        return true;
    }
};
```
You will find that the popup curiously doesn't show up. This is because `FLAlertLayer::show` by default adds the popup **to the current scene**, and a layer's `init` function is usually called right before leaving a scene, which makes the popup show on the previous scene instead of the current one. The solution to this is to first set the layer's `m_scene` member to the layer in `init` before calling `show`, as in:
```cpp
class $modify(MenuLayer) {
    bool init() {
        if (!MenuLayer::init())
            return false;
        
        auto alert = FLAlertLayer::create(
            "Title",
            "Hi mom!",
            "OK"
        );
        alert->m_scene = this;
        alert->show();

        return true;
    }
};
```
This will make the popup show correctly.

### Examples

 * [`ModSettingsPopup` in Geode, which uses `Popup`](https://github.com/geode-sdk/geode/blob/ui/loader/src/ui/internal/settings/ModSettingsPopup.hpp)
 * [Use of `createQuickPopup` within it](https://github.com/geode-sdk/geode/blob/ui/loader/src/ui/internal/settings/ModSettingsPopup.cpp#L156-L168)
