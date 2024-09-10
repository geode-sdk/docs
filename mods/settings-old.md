# Settings (Old)

> :warning: **Settings have been fully reworked**. See [the docs page for the new system](/mods/settings) for more information.

Geode mods can have settings, specified in `mod.json`. These settings are automatically saved and loaded, and an UI is produced for the user to edit them in-game through the mod's page.

Settings all have associated with them an ID, as well as a type and default value. There are a number of built-in types for common types of settings, but custom settings are supported aswell. All settings may also have a name and description. Some setting types also provide customizability over controls, such as whether to include a slider or not.

> :information_source: Settings are intended for values the user can customize in-game. If you're looking for just saving some data, [see the tutorial for saving data](/mods/savedata.md).

## Example

```json
{
    "settings": {
        "awesomeness-level": {
            "name": "Awesomeness level",
            "description": "How awesome the mod should be",
            "type": "int",
            "default": 0
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
        },
        "awesome-color": {
            "name": "Awesome color",
            "type": "color",
            "default": "#f438a2"
        }
    }
}
```

## Getting setting value

You can get the current value of a setting using the [getSettingValue](/classes/geode/Mod#getSettingValue) method. Note that the type you get the value as must match the value type of the setting type - see [Supported setting types](#supported-setting-types) for more info.

```cpp
auto value = Mod::get()->getSettingValue<int64_t>("my-int-setting");
```

## Detecting setting value changes

You can detect whenever the value of a setting is changed by using the [listenForSettingChanges] function. In most situations, you should call this function in an `$execute` block, so it gets enabled immediately when your mod is loaded. **The function will not be called on startup**, only when the value is changed afterwards. Note that the type you get the value as must match the value type of the setting type - see [Supported setting types](#supported-setting-types) for more info.

```cpp
$execute {
    listenForSettingChanges("my-float-setting", +[](double value) {
        // do something with the value
    });
}
```

## Supported setting types

*Tip: If you have our VSCode extension, you will get validation for these setting types*

Most of these can have a `default` key to set a default value.
```json
"my-setting": {
    "type": "string",
    "default": "John"
}
```

It can also be an object, for specifying the default value per platform:
```json
"my-setting": {
    "type": "string",
    "default": {
        "win": "Bill",
        "mac": "Steve",
        "android": "John"
    }
}
```

---

### `bool`

A simple boolean toggle.

### `int`

A 64-bit integer value. Value may be limited with the `min` and `max` properties. Available controls include a slider, arrows (buttons that increment/decrement by a predetermined amount), and a text input.

In code, the value type is `int64_t`.

### `float`

A 64-bit floating point (decimal) value. Value may be limited with the `min` and `max` properties. Available controls include a slider, arrows (buttons that increment/decrement by a predetermined amount), and a text input.

In code, the value type is `double`.

### `string`

A piece of text. Value may be constrained with the `match` key that specifies a regex that the string is matched against. \
The allowed characters can be set via the `filter` key, for example, `012345679,.` would only allow numbers and punctuation.

In code, the value type is `std::string`.

### `rgb` / `color`

An RGB color.

In code, the value type is `ccColor3b`.

### `rgba`

An RGBA color.

In code, the value type is `ccColor4b`.

### `path` / `file`

A file input, useful for example for configuration files. Controls include what file extension filters to show in the in-game file picker.

### `custom`

A custom setting; see the next section for more info.

## Custom settings

If the existing options aren't enough, mods may also provide custom settings. In this case, the mod needs to define two classes for Geode: a class that holds the setting's value, and a node class that represents it in the UI. As Geode does not know the UI or values for custom settings unless the mod is loaded, these settings can not be edited if the mod is disabled.

First, to declare a custom setting, the mod should specify that the setting's type is `custom` in the mod.json:

```json
{
    "settings": {
        "my-setting": {
            "type": "custom"
        }
    }
}
```

The first class that the mod needs to provide for the setting is one that inherits [SettingValue](/classes/geode/SettingValue). This class manages the current value of the setting as well as saving and loading that value.

```cpp
using namespace geode::prelude;

class MySettingValue : public SettingValue {
    // store the current value in some form.
    // this may be an enum, a class, or
    // whatever it is your setting needs -
    // you are free to do whatever!

public:
    // Make sure to have a public constructor!
    // Typically you always have these first two args,
    // since Mod::addCustomSetting expects them.
    MySettingValue(std::string const& key, std::string const& mod, T someValue)
      : SettingValue(key, mod), m_someMember(someValue) {}

    bool load(matjson::Value const& json) override {
        // load the value of the setting from json,
        // returning true if loading was succesful
    }
    bool save(matjson::Value& json) const override {
        // save the value of the setting into json,
        // returning true if saving was succesful
    }
    SettingNode* createNode(float width) override {
        return MySettingNode::create(width);
    }

    // getters and setters for the value
};
```

The second class the mod needs to provide is one that inherits [SettingNode](/classes/geode/SettingNode). This should be returned by the call to `MySettingValue::createNode`. It is used by Geode to display the setting's UI. Note that the node should only set the value of the setting when `commit` is called; this allows users to undo accidental changes.

The setting node is provided the current width of the setting layer available in its `init` function; the height of the setting may be as large or as small as it needs. **You must set the content size of the setting node** in order to let Geode know the height of your setting.

Whenever the value of the setting is changed using the node's controls, the node should call the `dispatchChanged` method to let Geode's UI know.

```cpp
using namespace geode::prelude;

class MySettingNode : public SettingNode {
protected:
    bool init(MySettingValue* value, float width) {
        if (!SettingNode::init(value))
            return false;

        // You may change the height to anything, but make sure to call
        // setContentSize!
        this->setContentSize({ width, 40.f });

        // Set up the UI. Note that Geode provides a background for the
        // setting automatically

        return true;
    }

    // Whenever the user interacts with your controls, you should call
    // this->dispatchChanged()

public:
    // When the user wants to save this setting value, this function is
    // called - this is where you should actually set the value of your
    // setting
    void commit() override {
        // Set the actual value

        // Let the UI know you have committed the value
        this->dispatchCommitted();
    }

    // Geode calls this to query if the setting value has been changed,
    // and those changes haven't been committed
    bool hasUncommittedChanges() override {
        // todo
    }

    // Geode calls this to query if the setting has a value that is
    // different from its default value
    bool hasNonDefaultValue() override {
        // todo
    }

    // Geode calls this to reset the setting's value back to default
    void resetToDefault() override {
        // todo
    }

    static MySettingNode* create(MySettingValue* value, float width) {
        auto ret = new MySettingNode();
        if (ret->init(value, width)) {
            ret->autorelease();
            return ret;
        }

        delete ret;
        return nullptr;
    }
};
```

The last this the mod needs to do is register their setting value class to Geode. This should be done immediately when the mod is loaded, to ensure that the value of the setting is loaded from disk on time. Registering can either be done using the raw [registerCustomSetting](/classes/geode/Mod#registerCustomSetting) method, or the [addCustomSetting](/classes/geode/Mod#addCustomSetting) convenience function.

```cpp
$on_mod(Loaded) {
    Mod::get()->addCustomSetting<MySettingValue>("my-setting", ...);

    // or, alternatively:
    Mod::get()->registerCustomSetting(
        "my-setting",
        std::make_unique<MySettingValue>("my-setting", Mod::get()->getID(), ...)
    );
}
```

You can find an example of custom settings [in the test dependency mod in Geode](https://github.com/geode-sdk/geode/blob/main/loader/test/dependency/main.cpp#L60-L145).
