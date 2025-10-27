# Saving data

Pretty much every mod will eventually enter the point where it needs to save some data on the user's machine, such as configuration settings or other user-customizable values. Geode has built-in support for **two kinds of savedata**: **user-editable settings** and **non-editable saved values**.

Settings are covered [in their own tutorial](/mods/settings.md) - **this tutorial is about non-user-editable save data**.

Unlike settings, save data does not need to be declared in `mod.json`, or anywhere else for that matter. Save data is automatically brought to life when you set it for the first time. Save data is, as the name implies, saved when the game is closed, and its previous state loaded back up the next time the game is opened.

You can save any type of value as long as it implements `matjson::Serialize` (see the [STL container implementations](https://github.com/geode-sdk/json/blob/main/include/matjson/stl_serialize.hpp) for an example).

## Setting & getting save data

You can set save data using [setSavedValue](/classes/geode/Mod#setSavedValue)

```cpp
Mod::get()->setSavedValue<float>("my-saved-value", .5f);
Mod::get()->setSavedValue<std::string>("my-other-value", "Garlagan!!!");

struct MyCustomSaveData {
    int x;
    int y;
};

template<>
struct matjson::Serialize<MyCustomSaveData> {
    static Result<MyCustomSaveData> fromJson(matjson::Value const& value) {
        GEODE_UNWRAP_INTO(int x, value["x"].asInt());
        GEODE_UNWRAP_INTO(int y, value["y"].asInt());
        return Ok(MyCustomSaveData{x, y});
    }

    static matjson::Value toJson(MyCustomSaveData const& value) {
        auto obj = matjson::Value();
        obj["x"] = value.x;
        obj["y"] = value.y;
        return obj;
    }
};

Mod::get()->setSavedValue("data", MyCustomSaveData { .x = 5, .y = 2 });
```

You can similarly get the current saved value using [getSavedValue](/classes/geode/Mod#getSavedValue). If the value has not been saved, a default value can be explicitly provided, or otherwise a default value will be created by invoking the save value's default constructor:

```cpp
auto value = Mod::get()->getSavedValue<float>("my-saved-value");
auto data = Mod::get()->getSavedValue<MyCustomSaveData>("data", MyCustomSaveData { .x = 0, .y = 0 });
```

As a useful little tip, `setSavedValue` returns the previous saved value, and if there is no previous saved value, default constructs it and returns that. You can use this knowledge, and combined with the knowledge that the default constructor for `bool` is false, to write one-time info popups using a single self-contained if statement:

```cpp
if (!Mod::get()->setSavedValue("shown-upload-guidelines", true)) {
    FLAlertLayer::create(
        "Upload guidelines",
        "You are only allowed to post pictures of cute catgirls",
        "OK"
    )->show();
}
```

This popup is only shown the first time the user enters it, and never again.

## Saving other data

Mods should save other data (files, backups, etc.) to their specific save directory, which they can access with `Mod::get()->getSaveDir()`. Using Geode's provided directories ensures that when the user uninstalls the mod, Geode can properly uninstall all its data. Geode's provided directories are also guaranteed to be readable and writable on all platforms.

It should be noted that a mod can have a good reason to save data elsewhere - for example a mod that saves created levels as individual files instead of CCLocalLevels would be justified in saving directly under the GD save folder instead of the mod save folder, since the data its saving is not related to the mod.

If you have data that the user should be able to edit (for example config files), these should go in the directory provided by `Mod::get()->getConfigDir()`. **Do not flood the main GD folder with config files or save data** - this will almost certainly get your mod rejected from the index unless you have a _very_ good reason for doing so!

