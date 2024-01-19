# Manual Hooks

Sometimes, you need to hook some platform specific function that would be a hassle to manually add to bindings - or you want to provide support for adding hooks through other means than `$modify`, such as an embedded scripting language. In this case, you can also manually add hooks in Geode using [`Mod::addHook`](/classes/geode/Mod#addHook).

## Example

The provided example hooks `MenuLayer::onNewgrounds` on Windows.

> :warning: This function can be hooked normally using `$modify` - this is just an example! **Do not do this!**

```cpp
void MenuLayer_onNewgrounds(MenuLayer* self, CCObject* sender) {
    log::info("Hook reached!");
    // You can call the original by calling it - in this case, since the 
    // original is in bindings, you can just call it the same way as you 
    // would with $modify
    // TODO: How to call original manually
    self->onNewgrounds(sender);
    log::info("After original!");
}

$execute {
    Mod::get()->hook(
        reinterpret_cast<void*>(geode::base::get() + 0x191E90), // address
        &MenuLayer_onNewgrounds, // detour
        "MenuLayer::onNewgrounds", // display name, shows up on the console
        tulip::hook::TulipConvention::Thiscall // calling convention
    );
}
```

## Hooking an imported function

This example shows how to hook a function that is linked to, for example through DLL imports. Geode comes with the `addresser` namespace for utilities related to figuring out the addresses of linked functions, including class methods and virtual functions.

```cpp
void myDrawCircle(const cocos2d::CCPoint& center, float radius, float angle, unsigned int segments, bool drawLineToCenter) {
    // Call the original
    cocos2d::ccDrawCircle(center, radius, angle, segments, drawLineToCenter);
    log::info("alright {}", radius);
}

$execute {
    Mod::get()->hook(
        reinterpret_cast<void*>(
            // All of this is to get the address of ccDrawCircle
            geode::addresser::getNonVirtual(
                // This is used because this function is overloaded,
                // otherwise just a regular function pointer would suffice (&foobar)
                geode::modifier::Resolve<const cocos2d::CCPoint&, float, float, unsigned int, bool>::func(&cocos2d::ccDrawCircle)
            )
        ),
        &myDrawCircle, // Our detour
        "cocos2d::ccDrawCircle", // Display name, shows up on the console
        tulip::hook::TulipConvention::Cdecl // Static free-standing cocos2d functions are cdecl
    );
}
```
