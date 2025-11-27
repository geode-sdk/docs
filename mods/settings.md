# Settings

> :warning: **Settings have been reworked. This docs page now details how to use the new system.** The docs page for the old system is [available elsewhere](/mods/settings-old).

Mods will often want to have properties that the user can configure, such as how often a backup should be made or if a menu redesign should be enabled. For this, Geode provides the **Settings** system for a standardized and simple way to create user-editable properties that are preserved between bootups.

All settings have an associated key, or ID. This is the key of the settings object defined in `mod.json`. Each setting is defined as a JSON object, which contains at least one key: `type`. This key defines the type of the setting, including a wide range of built-in types but also the `custom:` prefix for [defining your own types](#custom-settings).

> :information_source: Settings are intended for values the user can customize in-game. If you're looking for just saving some arbitary data in a quick cross-platform way, [see the tutorial for saving data](/mods/savedata.md).

![An image showcasing the settings popup for the BetterEdit mod by HJfod](/assets/settings/popup.png)

## Using settings

> :information_source: It is heavily recommended to have [the Geode VS Code Extension](/getting-started/ide-setup) installed, as it gives you automatic code completion and checking for settings in `mod.json`.

Adding new settings to your mod happens through `mod.json`. Define the `settings` key as an object and then list out the settings you want to have: 

```json
{ // mod.json
    "geode": "3.5.0",
	"id": "me.my-mod",
	"name": "My Awesome Mod!",
	"version": "1.0.0",
    "settings": {
        "awesome-title": {
            "type": "title",
            "name": "Awesome Settings"
        },
        "awesomeness-level": {
            "name": "Awesomeness level",
            "description": "How awesome the mod should be",
            "type": "int",
            "default": 0
        },
        "awesome-color": {
            "name": "Awesome color",
            "type": "color",
            "default": "#f438a2"
        },
        "misc-title": {
            "type": "title",
            "name": "Miscellanious"
        },
        "slider": {
            "name": "Slider!! Weeeee",
            "type": "float",
            "default": 0,
            "min": -10,
            "max": 10,
            "control": {
                "slider": true
            }
        }
    }
}
```

Once you have defined your settings, you can use them in your mod via the [`Mod::getSettingValue`](/classes/geode/Mod#getSettingValue) function:

```cpp
auto myBool = Mod::get()->getSettingValue<bool>("my-bool-setting");
auto myInt  = Mod::get()->getSettingValue<int64_t>("my-int-setting");
```

You can detect whenever the value of a setting is changed by using the [listenForSettingChanges] function. In most situations, you should call this function in an `$execute` block, so it gets enabled immediately when your mod is loaded. **The function will not be called on startup**, only when the value is changed afterwards. Note that the type you get the value as must match the value type of the setting type - if you are using a custom setting, make sure to specialize `geode::SettingTypeForValueType<T>`.

```cpp
#include <Geode/loader/SettingV3.hpp>

using namespace geode::prelude;

$execute {
    listenForSettingChanges("my-float-setting", [](double value) {
        // do something with the value
    });
}
```

## Setting types

Every setting must have at least one property: `type`. This defines what kind of setting it is; there are a bunch of built-in settings for handling common data types, but you can also create custom setting types via the `custom:<type-name>` syntax.

All setting types, including custom ones if they opt-in, share a common set of base keys that can be used to control the UI and the setting:

| Key    | Type | Description                               |
|--------|------|-------------------------------------------|
| `type` | Valid setting type | The type of the setting; see below for possible types. This is required for all settings! |
| `default` | (Depends on the type) | The default value for the setting. This is required for all settings except for `title` (and custom settings that opt-out). May be specified per-platform, i.e. `"default": { "win": ..., "mac": ..., "android": ... }` |
| `name` | String | The name of the setting. This is optional, but heavily recommended. This name is shown in the UI |
| `description` | String | A description for what the setting is for. This is optional. This adds a description popup button to the UI |
| `requires-restart` | Boolean | Whether this setting requires the game to be restarted when its value is changed  |
| `platforms` | Array | List of platforms this setting is available on, i.e. `["win", "android"]` |
| `enable-if` | [`enable-if` Scheme](#enable-if) | A way to disable settings based on other settings; see [Enable If Schemes](#enable-if) for more |
| `enable-if-description` | String | Human-readable description for what the `enable-if` scheme does. If not provided, Geode will try to synthesize a legible one from the `enable-if` scheme itself. However, specifying a custom description is heavily recommended if you use saved values, or have more complex schemes |

---

### Title (`title`)

![An image showcasing a basic title setting](/assets/settings/title.png)

Title settings are cosmetic settings that can be used to group other settings.

> Unlike all other built-in settings, title settings only have a `name` and `description` key available, and lack the rest of the base keys such as `default`, `restart-required` etc.

```json
"title-setting-example": {
    "type": "title",
    "name": "Groovy Settings",
    "description": "These settings control how groovy you are"
}
```

---

### Boolean (`bool`)

![An image showcasing a basic boolean setting](/assets/settings/bool.png)

Boolean settings are a simple toggle for whether the setting is enabled or not.

```json
"bool-setting-example": {
    "type": "bool",
    "name": "Enable Groovy Mode",
    "description": "Bring some real action to this lame ass party",
    "default": true
}
```

```cpp
auto value = Mod::get()->getSettingValue<bool>("bool-setting-example");
```

---

### Integer (`int`)

![An image showcasing a basic integer setting](/assets/settings/int.png)

Integer settings are a whole number. By default, they have a slider, arrows to increase by 1/-1, and a text input.

```json
"int-setting-example": {
    "type": "int",
    "name": "Groovy Level",
    "description": "How groovy your grooving is",
    "default": 5,
    // Minimum value
    "min": 0,
    // Maximum value
    "max": 10,
    // Define the controls for this setting
    "controls": {
        // Enable the small (green) arrow controls
        "arrows": true,
        // Control how much the small (green) arrows should increment/decrement 
        // the setting's value when clicked
        "arrow-step": 1,
        // Enable the secondary (pink) arrow controls
        "big-arrows": false,
        // Control how much the secondary (pink) arrows should increment/
        // decrement the setting's value when clicked
        "big-arrow-step": 5,
        // Enable the slider
        "slider": true,
        // Control the slider's snap step size
        "slider-step": 1,
        // Enable the text input
        "input": false
    }
}
```

```cpp
auto value = Mod::get()->getSettingValue<int64_t>("int-setting-example");
```

---

### Float (`float`)

![An image showcasing a basic float setting](/assets/settings/float.png)

Float settings are just like int settings, but for floats!

```json
"float-setting-example": {
    "type": "float",
    "name": "Groove Depth",
    "description": "How deep your grooving is",
    "default": 1,
    "min": 0.5,
    "max": 2.5,
    // Define the controls for this setting
    "controls": {
        // Enable the small (green) arrow controls
        "arrows": true,
        // Control how much the small (green) arrows should increment/decrement 
        // the setting's value when clicked
        "arrow-step": 0.1,
        // Enable the secondary (pink) arrow controls
        "big-arrows": false,
        // Control how much the secondary (pink) arrows should increment/
        // decrement the setting's value when clicked
        "big-arrow-step": 0.5,
        // Enable the slider
        "slider": true,
        // Control the slider's snap step size
        "slider-step": 0.1,
        // Enable the text input
        "input": false
    }
}
```

```cpp
auto value = Mod::get()->getSettingValue<double>("float-setting-example");
```

---

### String (`string`)

![An image showcasing a basic string setting](/assets/settings/string.png)
![An image showcasing a basic enum string setting via the `one-of` property](/assets/settings/string-one-of.png)

String settings are simple strings that can be controlled by character limits or a regex pattern. These can also be used to create **enum settings** via the `one-of` key.

```json
"string-setting-example": {
    "type": "string",
    "name": "Groovy Name",
    "default": "Stephen",
    // A regex pattern the setting must match
    "match": "[A-Z][a-z]+",
    // A list of characters this setting can consist of, same as the 
    // CCTextInputNode character filter
    "filter": "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ",
    // A list of possible values. Turns the setting into an enum, where the 
    // user can simply use arrows to navigate the list of values
    "one-of": ["Stephen", "George", "Sarah", "Tiffany"]
}
```

```cpp
auto value = Mod::get()->getSettingValue<std::string>("string-setting-example");
```

---

### File (`file`)

![An image showcasing a basic file setting](/assets/settings/file.png)

File settings allow the user to select a file or path, such as the path to a config file, or an external application on their device.

The default value for file settings support a set of known path prefixes to allow referencing certain files, such as `{gd_dir}/GeometryDash.exe`. The list of path prefixes are:

| Variable | Path |
|----------|------|
| `{gd_dir}` | The main Geometry Dash directory (which contains `GeometryDash.exe` on Windows) |
| `{gd_save_dir}` | The Geometry Dash save directory |
| `{gd_resources_dir}` | The Geometry Dash resources directory |
| `{mod_config_dir}` | The Geode-provided config directory for this mod |
| `{mod_save_dir}` | The save directory for this mod |
| `{mod_resources_dir}` | The resources directory for this mod |
| `{temp_dir}` | The Geode directory for temporary files |

> :warning: Note that there is an alternative name for this type - `path` - however, **the `path` type has been deprecated**. Use either the `file` or `folder` type.

> :information_source: Some mods set the `default` value to be a description for what kind of file to pick instead of a path, as in `"default": "Please pick an image file."`. This is allowed, and Geode will even try its best to detect this and show the text in gray (like the built-in description), although it should be noted that this is in the end a hack and might cause unexpected issues.

```json
"file-setting-example": {
    "type": "file",
    "name": "Groovy Executable Path",
    "default": {
        "win": "C:/Program Files/Groovy/groovy.exe",
        "mac": "",
        "android": "{gd_dir}/groovy.so"
    },
    "control": {
        // The dialog to show when the user clicks the file selection button in the 
        // UI. Either "save" or "open" - defaults to "open"
        "dialog": "open",
        // Configure file filters in the open dialog. An "All (*.*)" filter is 
        // added implicitly
        "filters": [
            {
                "files": ["*.exe", "*.so", "*.dmg"],
                "description": "Executable programs"
            }
        ]
    }
}
```

```cpp
auto value = Mod::get()->getSettingValue<std::filesystem::path>("file-setting-example");
```

---

### Folder (`folder`)

![An image showcasing a basic folder setting](/assets/settings/folder.png)

Folder settings, similar to file settings, allow the user to select a folder or path, such as the directory to put exported files into.

The default value for folder settings support the same set of known path settings as file settings, like `{gd_dir}`.

> :warning: Note that there is an alternative name for this type - `path` - however, **the `path` type has been deprecated**. Use either the `file` or `folder` type.

```json
"folder-setting-example": {
    "type": "folder",
    "name": "Groovy Resources Directory",
    "default": "{gd_dir}/Resources"
}
```

```cpp
auto value = Mod::get()->getSettingValue<std::filesystem::path>("folder-setting-example");
```

---

### Color (`color` / `rgb` / `rgba`)

![An image showcasing a basic RGB setting](/assets/settings/rgb.png)
![An image showcasing a basic RGBA setting](/assets/settings/rgba.png)

Color settings prompt the user to pick a specified color. The `color` and `rgb` types are RGB-only, whereas the `rgba` type is an RGBA color.

```json
"rgb-setting-example": {
    "type": "rgb",
    "name": "Groovy Hair Color",
    "default": "#a8f02d"
}
"rgba-setting-example": {
    "type": "rgba",
    "name": "Groovy Hair (Possibly Bald) Color",
    "default": [143, 253, 89, 255]
}
```

```cpp
auto rgb  = Mod::get()->getSettingValue<cocos2d::ccColor3B>("rgb-setting-example");
auto rgba = Mod::get()->getSettingValue<cocos2d::ccColor4B>("rgba-setting-example");
```

---

## Custom Settings

Mods can also specify custom setting types. Unlike the old system, where custom settings where defined per-setting, under the new system you can reuse the same custom setting type for multiple settings.

First, to define a setting as using a custom type in your JSON, simply use the `custom:` prefix:

```json
"my-setting": {
    "type": "custom:my-awesome-type",
    "name": "My Awesome Enum!",
    "default": "my-custom-enum-value"
}
```

Custom settings by default have all the same [base properties](#setting-types) as all other settings, such as `"name"`, `"description"`, and even properties like `"enable-if"`. These can be opted out of though, as shown in the second example for a button setting.

> :information_source: Custom settings may also use types from other mods, such as `"type": "custom:hjfod.more-setting-types/player-icon"`. Make sure to include the mod in your [dependencies](/mods/dependencies) if you do this!

Custom setting types must define two classes: a `SettingV3`-based definition class, and a `SettingNodeV3`-based node for the UI.

**As an example, here's a simple enum-based custom setting for picking between a list of options**

![An image showcasing the custom enum setting in action in Geode](/assets/settings/custom-enum.png)

```cpp
#include <Geode/loader/SettingV3.hpp>
#include <Geode/loader/Mod.hpp>

using namespace geode::prelude;

// Settings should have some way to store their value. Here we use an enum, but 
// you can of course use a more complicated type aswell!
// The main point is that the type *must implement `matjson::Serialize`*.
enum class MyCustomEnum {
    ValidEnumValue,
    OtherValidEnumValue,
};

// The type of the setting must implement `matjson::Serialize` in order to use 
// `SettingBaseValueV3`
template <>
struct matjson::Serialize<MyCustomEnum> {
    // See https://github.com/geode-sdk/test-mod/blob/main/src/MyCustomSettingV3.cpp#L12-L30 
    // for the definition
};

// This is the most important part of the setting - this actually defines what 
// properties the setting can have in `mod.json`, as well as keeps track of the 
// setting's current value.
// You can inherit directly for `SettingV3`, but for most cases you'll want to 
// inherit from `SettingBaseValueV3` instead, as it automatically provides the 
// `"default"` property, defines most of the required virtuals from `SettingV3`, 
// and provides getters and setters for the current value.
// Using `SettingBaseValueV3` requires the value type to implement 
// `matjson::Serialize`.
// For an example on how to inherit directly from `SettingV3` (for example for 
// cosmetic purposes), see below for the `MyButtonSettingV3` example
class MyCustomSettingV3 : public SettingBaseValueV3<MyCustomEnum> {
protected:
    // We don't need to store the value nor the default value ourselves, as 
    // `SettingBaseValueV3` handles that for us :3
    // However, we will add an additional property for the sake of showcasing 
    // how one would deal with that. This property simply marks whether the 
    // setting should be "splorgy", which by default is false.
    bool m_splorgy = false;

public:
    // All settings must provide a `parse` function for parsing the setting 
    // from its definition in `mod.json`. The signature must match this exactly, 
    // although the return type should point to a `SettingV3`-derivative rather 
    // than `std::shared_ptr<SettingV3>` directly
    static Result<std::shared_ptr<SettingV3>> parse(
        // The key of the setting, as defined in `mod.json`
        std::string const& key,
        // The mod ID this setting is being parsed for
        std::string const& modID,
        // The actual definition of the setting (an object)
        matjson::Value const& json
    ) {
        auto res = std::make_shared<MyCustomSettingV3>();

        // It is recommended to use the JSON checking utilities in Geode for 
        // actually parsing the setting, as this deals with type & error 
        // handling really nicely
        auto root = checkJson(json, "MyCustomSettingV3");

        // You should basically always call `parseBaseProperties`, as this will 
        // deal with all of the base keys like `"name"`, `"description"`, etc. 
        // If you derive from `SettingBaseValueV3`, this will also parse the 
        // `"default"` key appropriately.
        // There is also an overload for parsing directly from JSON if you don't 
        // want to use Geode's JSON checking utilities, as well as a fourth bool 
        // argument for parsing just the `"name"` and `"description"` properties 
        // if your setting is only cosmetic.
        
        // If your setting doesn't even have a name or description, you must at 
        // the very least call `res->init(key, modID)`!
        res->parseBaseProperties(key, modID, root);

        // This is where we parse any properties specific to this setting. In 
        // our case, that will be splorgyness, which is optionally defined:
        root.has("splorgy").into(res->m_splorgy);

        // This logs a warning into the console if the object contains keys you 
        // haven't defined. You can remove it if you don't like logspam
        root.checkUnknownKeys();

        // Return the resulting instance, or an Err if the JSON parsing failed
        return root.ok(std::static_pointer_cast<SettingV3>(res));
    }
    
    // This is defined at the end of the file, as it needs to know the  
    // definition of the `MyCustomSettingNodeV3` class
    SettingNodeV3* createNode(float width) override;
};

// If you want to be able to use `Mod::getSettingValue` with your custom setting, 
// specialize this type!
// Afterwards, you can simply do `Mod::get()->getSettingValue<MyCustomEnum>("my-setting")`
template <>
struct geode::SettingTypeForValueType<MyCustomEnum> {
    using SettingType = MyCustomSettingV3;
};

// The second most important part of the setting is defining the node for the 
// UI. Again, here we use a helper class, which automatically defines a bunch 
// of the required virtual functions for us. Note that using 
// `SettingValueNodeV3` requires the setting type to implement 
// `SettingBaseValueV3`!
class MyCustomSettingNodeV3 : public SettingValueNodeV3<MyCustomSettingV3> {
protected:
    // If we use `SettingValueNodeV3`, we once again don't need to track the 
    // current value ourselves - we just use the `getValue` and `setValue` 
    // functions!
    std::vector<CCMenuItemToggler*> m_toggles;

    bool init(std::shared_ptr<MyCustomSettingV3> setting, float width) {
        if (!SettingValueNodeV3::init(setting, width))
            return false;
        
        // Create toggles for each of our enum options
        for (auto value : {
            std::make_pair(MyCustomEnum::ValidEnumValue, "GJ_starsIcon_001.png"),
            std::make_pair(MyCustomEnum::OtherValidEnumValue, "currencyOrbIcon_001.png"),
        }) {
            auto offSpr = CCSprite::createWithSpriteFrameName(value.second);
            offSpr->setOpacity(90);
            auto onSpr = CCSprite::createWithSpriteFrameName(value.second);
            auto toggle = CCMenuItemToggler::create(
                offSpr, onSpr, this, menu_selector(MyCustomSettingNodeV3::onToggle)
            );
            toggle->m_notClickable = true;
            toggle->setTag(static_cast<int>(value.first));
            m_toggles.push_back(toggle);
            
            // You should add all of your menu components to the menu returned by 
            // `this->getButtonMenu()`
            this->getButtonMenu()->addChild(toggle);
        }

        // You should resize the menu to fit your content, as this will 
        // automatically resize the name label to give it the right amount of 
        // space
        this->getButtonMenu()->setContentWidth(40);
        this->getButtonMenu()->setLayout(RowLayout::create());

        // Remember to always call `updateState` at the end of your init!
        this->updateState(nullptr);
        
        return true;
    }
    
    void updateState(CCNode* invoker) override {
        // Remember to always call the base class's `updateState`!
        SettingValueNodeV3::updateState(invoker);

        // If your setting supports "enable-if" schemes, you should make sure 
        // that the UI is updated to reflect that. If `shouldEnable` returns 
        // false, all toggles and buttons should be grayed out and disabled!
        auto shouldEnable = this->getSetting()->shouldEnable();

        // Update the state of all the toggles
        for (auto toggle : m_toggles) {
            // Enable the toggle if it corresponds to the current value
            toggle->toggle(toggle->getTag() == static_cast<int>(this->getValue()));

            // Gray out the toggle if it should be disabled
            toggle->setEnabled(shouldEnable);
            toggle->setCascadeColorEnabled(true);
            toggle->setCascadeOpacityEnabled(true);
            toggle->setOpacity(shouldEnable ? 255 : 155);
            toggle->setColor(shouldEnable ? ccWHITE : ccGRAY);
        }
    }
    void onToggle(CCObject* sender) {
        this->setValue(
            // Get the value of the toggle
            static_cast<MyCustomEnum>(sender->getTag()),
            // `setValue` eventually calls `updateState`, so it needs to know 
            // what the invoker of the state update was
            static_cast<CCNode*>(sender)
        );
    }

public:
    static MyCustomSettingNodeV3* create(std::shared_ptr<MyCustomSettingV3> setting, float width) {
        auto ret = new MyCustomSettingNodeV3();
        if (ret->init(setting, width)) {
            ret->autorelease();
            return ret;
        }
        delete ret;
        return nullptr;
    }
};

// Define the node creation function
SettingNodeV3* MyCustomSettingV3::createNode(float width) {
    return MyCustomSettingNodeV3::create(
        std::static_pointer_cast<MyCustomSettingV3>(shared_from_this()),
        width
    );
}

// When the mod is loaded, you need to register your setting type
$execute {
    // You can also handle the errors, but if this fails, the game is probably about to crash anyway
    (void)Mod::get()->registerCustomSettingType("my-awesome-type", &MyCustomSettingV3::parse);
}
```

If you want to store a more complex value (e.g. a string or a custom class/struct), make sure the value type is copyable, comparable, and has a JSON serialization implementation. This can be done as follows:

```cpp
struct MyComplexSettingValue {
    // In case you're wondering why we can't just define a custom setting with 
    // `std::string` as the value type: that would conflict with the built-in 
    // string setting, so `Mod::getSettingValue<std::string>()` would  
    // unexpectedly fail
    std::string value;

    // Make sure your value type is comparable
    bool operator==(MyComplexSettingValue const& other) const = default;

    // If your value type is a thin wrapper around another type, you can allow 
    // implicit conversions from your type to the wrapped type
    operator std::string() const {
        return value;
    }

    MyComplexSettingValue() = default;
    MyComplexSettingValue(std::string_view value) : value(value) {}

    // Setting values must be copyable!
    MyComplexSettingValue(MyComplexSettingValue const&) = default;
};

// You'll have to manually implement JSON serialization for the value
template<>
struct matjson::Serialize<MyComplexSettingValue> {
    // Serialize the value into JSON. In this case, we just return the value, 
    // as strings are inherently JSON-serializable
    static matjson::Value toJson(MyComplexSettingValue const& settingValue) {
        return settingValue.value;
    }
    // Deserialize the value from JSON, again taking advantage of strings being 
    // inherently JSON-serializable
    static Result<MyComplexSettingValue> fromJson(matjson::Value const& json) {
        GEODE_UNWRAP_INTO(auto str, json.asString());
        return Ok(MyComplexSettingValue(str));
    }
};
```

Custom settings do not necessarily need to inherit from the `SettingValueNodeV3<T>` helper. For example, if you wanted to make a non-conventional setting, like show a non-interactive graphic or button, you can inherit directly from `SettingV3`.

**As an example, here's a custom setting type for a button that opens a website when clicked:**

![An image showcasing the custom button setting in action in Geode](/assets/settings/custom-button.png)

```cpp
#include <Geode/loader/SettingV3.hpp>
#include <Geode/loader/Mod.hpp>

// If you use PCH these are most likely not necessary
#include <Geode/binding/ButtonSprite.hpp>
#include <Geode/binding/CCMenuItemSpriteExtra.hpp>
#include <Geode/binding/FLAlertLayer.hpp>

using namespace geode::prelude;

// Inherit from SettingV3 directly over SettingBaseValueV3, as our setting 
// doesn't actually control any value, but it just a simple button
class MyButtonSettingV3 : public SettingV3 {
public:
    // Once again implement the parse function
    static Result<std::shared_ptr<SettingV3>> parse(std::string const& key, std::string const& modID, matjson::Value const& json) {
        auto res = std::make_shared<MyButtonSettingV3>();
        auto root = checkJson(json, "MyButtonSettingV3");

        // `parseBaseProperties` parses all base properties, including 
        // `requires-restart` - which does not really make sense in our use case, 
        // as there's nothing in this setting that could require a restart.
        // So, we instead parse the base properties that actually apply to this 
        // setting manually:
        res->init(key, modID, root);
        res->parseNameAndDescription(root);
        res->parseEnableIf(root);
        
        root.checkUnknownKeys();
        return root.ok(std::static_pointer_cast<SettingV3>(res));
    }

    // Since there's no data to save or load, these can just return true
    // Although you could use these for example to store how many times the 
    // button has been clicked 
    bool load(matjson::Value const& json) override {
        return true;
    }
    bool save(matjson::Value& json) const override {
        return true;
    }

    // This setting can't ever have anything but the default value, as it has 
    // no value
    bool isDefaultValue() const override {
        return true;
    }
    void reset() override {}

    // Once again defined out-of-line
    SettingNodeV3* createNode(float width) override;
};

// We are inheriting from `SettingNodeV3` directly again, as we don't need the 
// boilerplate `SettingValueNodeV3` fills in for us because our setting has no 
// value!
class MyButtonSettingNodeV3 : public SettingNodeV3 {
protected:
    ButtonSprite* m_buttonSprite;
    CCMenuItemSpriteExtra* m_button;

    bool init(std::shared_ptr<MyButtonSettingV3> setting, float width) {
        if (!SettingNodeV3::init(setting, width))
            return false;
    
        // We just create the button and add it to the setting's menu
        
        m_buttonSprite = ButtonSprite::create("Click Me!", "goldFont.fnt", "GJ_button_01.png", .8f);
        m_buttonSprite->setScale(.5f);
        m_button = CCMenuItemSpriteExtra::create(
            m_buttonSprite, this, menu_selector(MyButtonSettingNodeV3::onButton)
        );
        this->getButtonMenu()->addChildAtPosition(m_button, Anchor::Center);
        this->getButtonMenu()->setContentWidth(60);
        this->getButtonMenu()->updateLayout();

        this->updateState(nullptr);
        
        return true;
    }
    
    void updateState(CCNode* invoker) override {
        SettingNodeV3::updateState(invoker);

        // In case there's an "enable-if" condition on this button, for example 
        // if this played a test notification in a notification mod and you 
        // want to disable the button if notifications are disabled entirely
        auto shouldEnable = this->getSetting()->shouldEnable();
        m_button->setEnabled(shouldEnable);
        m_buttonSprite->setCascadeColorEnabled(true);
        m_buttonSprite->setCascadeOpacityEnabled(true);
        m_buttonSprite->setOpacity(shouldEnable ? 255 : 155);
        m_buttonSprite->setColor(shouldEnable ? ccWHITE : ccGRAY);
    }
    void onButton(CCObject*) {
        FLAlertLayer::create(
            "Hello",
            fmt::format(
                "You clicked the button for setting <co>{}</c>!",
                this->getSetting()->getDisplayName()
            ),
            "OK"
        )->show();
    }

    // Both of these can just be no-ops, since they make no sense for our 
    // setting as it's just a button
    void onCommit() override {}
    void onResetToDefault() override {}

public:
    static MyButtonSettingNodeV3* create(std::shared_ptr<MyButtonSettingV3> setting, float width) {
        auto ret = new MyButtonSettingNodeV3();
        if (ret->init(setting, width)) {
            ret->autorelease();
            return ret;
        }
        delete ret;
        return nullptr;
    }

    // Both of these can just return false, since they make no sense for our 
    // setting as it's just a button
    bool hasUncommittedChanges() const override {
        return false;
    }
    bool hasNonDefaultValue() const override {
        return false;
    }

    // This is not necessary, but it makes it so you don't have to do the 
    // pointer cast every time you want to access the properties of the button 
    // setting
    std::shared_ptr<MyButtonSettingV3> getSetting() const {
        return std::static_pointer_cast<MyButtonSettingV3>(SettingNodeV3::getSetting());
    }
};

// Create node as before
SettingNodeV3* MyButtonSettingV3::createNode(float width) {
    return MyButtonSettingNodeV3::create(
        std::static_pointer_cast<MyButtonSettingV3>(shared_from_this()),
        width
    );
}

// Register as before
$execute {
    (void)Mod::get()->registerCustomSettingType("my-button-type", &MyButtonSettingV3::parse);
}
```

## Enable If

![An image showcasing a setting disabled due to an `enable-if` condition](/assets/settings/enable-if.png)

Starting with 3.5.0, settings may define an `enable-if` scheme for conditionally disabling them based on the values of other settings, or the state of the mod using saved values. For example, if you have a toggle that enables a feature and then a text input for fine-tuning that feature, you could use `enable-if` to disable the input when the toggle is not enabled.

The simplest `enable-if` scheme looks like this:
```json
"enable-if": "other-settings-key"
```
By default, all identifiers in an `enable-if` scheme are assumed to be setting names. However, you can also control a setting based on a (boolean) saved value with the `saved:` prefix:
```json
"enable-if": "saved:my-boolean-saved-value"
```
You can also use `setting:` to explicitly specify that you are referencing a setting, i.e. `setting:other-settings-key`.

You can also refer to settings and saved values from other mods:
```json
"enable-if": "hjfod.betteredit/larger-color-menu"
```

`enable-if` also supports basic boolean operations, including and (`&&`), or (`||`), parentheses (`()`), and negation (`!`):
```json
"enable-if": "first-toggle && !second-toggle"
```

> :warning: Note that currently **only references to boolean settings and saved values are supported inside `enable-if` schemes**. Support for comparing integer values may be added in the future, but it's unlikely as we don't want to introduce a whole programming language just for controlling if a setting is enabled or not.

## Migrating from v2

Migrating from the old settings system to the new one for 90% of mods is as simple as recompiling. By default, the new system is **fully backwards-compatible**, so APIs such as `Mod::getSettingValue` work the exact same as before, and the syntax for existing setting types is the same. In addition, now-deprecated features such as bare `custom` settings and the `path` type **will continue to work until Geode 4.0.0**, when they will be removed. This is to give developers a good grace period to migrate over to the new system.

A list of the major changes in the new system are:
 * The `path` setting type has been split to the `file` or `folder` types
 * A new `title` type has been added for creating titles
 * The `custom` type has been deprecated in favor of the `custom:` prefix
 * The basic properties like `"name"` and `"description"` are now available by default for all settings, including custom ones
 * The `SettingValue` class has been deprecated in favor of the `SettingV3` class
 * The `SettingNode` class has been deprecated in favor of the `SettingNodeV3` class
 * The `Mod::getSetting` and `Mod::getSettingDefinition` functions have been replaced with `Mod::getSettingV3`
 * The `SettingValueSetter` class has been replaced with `SettingTypeForValueType`

Most of the migration work comes from custom settings. As the classes used for settings have been changed, and moreso as custom settings have moved away from customizing specific settings over to custom setting types, they may require a lot more refactoring to be fully migrated over.

If you have used custom settings for the purpose of creating titles, you should use [the new `title` type](#title-title) instead.

**Porting custom settings in general works like this:**
1. Replace `"type": "custom"` with `"type": "custom:my-type-name"` in `mod.json`. If you have a bunch of similar custom settings, consider making them all the same type and customizing via adding more properties to the JSON!
2. Create a new `SettingV3`-based class for all your custom types, following [the tutorial for making custom settings](#custom-settings). Most of the functions in `SettingValue` have direct equivalents in `SettingV3`, so it should be easy to figure out how to convert stuff over from your old classes over to the new system.
3. Create a new `SettingNodeV3`-based class for the nodes. Do take note that while `SettingNode` is a blank canvas to put whatever in, **`SettingNodeV3` already gives you the title label and description button**, among other shared UI components, so all you need to care about is adding your own controls to the menu returned by `SettingNodeV3::getButtonMenu`.

Unless your custom settings are particularly complex, **it is recommended to just rewrite them from scratch**. This will take a bit of effort, but should be pretty easy [if you follow the guide for creating custom settings](#custom-settings). It is also just good practice in general to (if possible) rewrite parts of your codebase every now and then to make sure everything is as refined as possible.

It is heavily recommended to follow the practices laid out in the [Custom Settings part of this tutorial](#custom-settings) for setting nodes, as this results in conventional, easy-to-use and easy-to-maintain UIs. However, if you do have a reason to make a setting that has unconventional UI, you can of course always hide the name label and do what you want.

