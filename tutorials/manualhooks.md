# Manual Hooks

Incase you need to hook some platform specific function that would otherwise be a hassle to add to the bindings, you can use `Mod::addHook`. For functions from GD itself this should be avoided.

## Example

Say I wanted to hook `MenuLayer::onNewgrounds` on windows (**this function can be hooked normally using [modify](#), this is just an example**)

```cpp
void MenuLayer_onNewgrounds(MenuLayer* self, CCObject* sender) {
    log::info("Hook reached!");
    self->onNewgrounds(sender);
    log::info("After original!");
}

$execute {
    Mod::get()->addHook(
        reinterpret_cast<void*>(geode::base::get() + 0x191E90), // address
        &MenuLayer_onNewgrounds, // detour
        "MenuLayer::onNewgrounds", // display name, shows up on the console
        tulip::hook::TulipConvention::Thiscall // calling convention
    );
}
```

## Hooking some imported function

Say you want to hook some function thats available to you through some dll import for example, then you can use geode's Addresser for this.

```cpp
void myDrawCircle(const cocos2d::CCPoint& center, float radius, float angle, unsigned int segments, bool drawLineToCenter) {
    // calling orig
    cocos2d::ccDrawCircle(center, radius, angle, segments, drawLineToCenter);
    log::info("alright {}", radius);
}

$execute {
    Mod::get()->addHook(
        reinterpret_cast<void*>(
            // all of this is to get the address
            geode::addresser::getNonVirtual(
                // this is used because this function is OVERLOADED,
                // otherwise just a regular function pointer would suffice (&foobar)
                geode::modifier::Resolve<const cocos2d::CCPoint&, float, float, unsigned int, bool>::func(&cocos2d::ccDrawCircle)
            )
        ),
        &myDrawCircle, // detour
        "cocos2d::ccDrawCircle", // display name, shows up on the console
        tulip::hook::TulipConvention::Cdecl // static cocos2d functions are cdecl
    );
}
```