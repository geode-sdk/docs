# Migrating your mods from the traditional

As Geode is currently incompatible with traditional mods, migration of old mods is a must in order to use them with Geode. This page shows how to do just that.

## Migrating from gd.h and cocos-headers

This one may be one of the most tedious parts. First of all, all of the GD classes are removed from the `gd` namespace. And since older gd.h's use the `m_tVariable` convention with t being the type identifier, all of those variables will give errors when given Geode's `m_variable` convention. Let's see an example of this.

Here is a code snippet from HJFod's [BetterEdit v4](https://github.com/HJfod/BetterEdit/blob/v4-min/hooks/EditorUI.cpp#L39-L53):

```cpp
CCPoint getShowButtonPosition(EditorUI* self) {
    auto winSize = CCDirector::sharedDirector()->getWinSize();
    auto ratio = winSize.width / winSize.height;

    if (ratio > 1.5f)
        return {
            self->m_pTrashBtn->getPositionX() + 50.0f,
            self->m_pTrashBtn->getPositionY()
        };
    
    return {
        self->m_pPlaybackBtn->getPositionX() + 45.0f,
        self->m_pPlaybackBtn->getPositionY()
    };
}
```
This code uses the `m_pTrashBtn` and the `m_pPlaybackBtn` members from `EditorUI`, a class from gd.h. These members correspond to `m_trashBtn` and `m_playbackBtn` in Geode respectively. It is not the best way to do it, but one can use regex `m_(?:ob|[a-z])([A-Z])` to find most of the variables that use the old convention.

Cocos-headers don't need any extra work, since the cocos headers in Geode are based on that repository with changes needed for the Geode codebase.


## Migrating the hooks

### Bare Minhook

This is the most common used method. Minhook hooks have a static function pointer in which the trampoline is stored (often prefixed with `_O`) and the hook itself (often prefixed with `_H`). Calling the original is done by calling the trampoline stored in the function pointer.

The Minhook is first initialized with `MH_Initialize`, then hooks are added with `MH_CreateHook` with parameters address, the address of the hook function, and the address of the static function pointer, then `MH_EnableHook` is called to enable the hooks themselves.

```cpp
bool (__thiscall* MenuLayer_init_O)(gd::MenuLayer* self);
bool __fastcall MenuLayer_init_H(gd::MenuLayer* self, void*) {
    if (!MenuLayer_init_O(self)) return false;
    return true;
}
// ...
MH_Initialize();
MH_CreateHook((void*)base + 0x1907b0, (void*)&MenuLayer_init_H, (void**)&MenuLayer_init_O);
MH_EnableHook(MH_ALL_HOOKS);
```

### MAT dash

This is also used by some people, mainly by newer traditional modders. Compared to other methods, MAT dash uses the return type of the hook to infer the calling convention (example: `matdash::cc::thiscall<void>`) unless it's membercall, a GD specific calling convention, which needs to be removed for migration. The original is called using `matdash::orig` with a template parameter of a reference to the hook, the parameters are the same with the hook itself.

MAT dash hooks are added with `matdash::add_hook` with a template parameter of a reference to the hook function and a parameter of the address.

```cpp
bool MenuLayer_init(gd::MenuLayer* self) {
    if (!matdash::orig<&MenuLayer_init>(self)) return false;
    return true;
}
// ...
matdash::add_hook<&MenuLayer_init>(base + 0x1907b0)
```

### GDMake

No one will need this section but I'm adding it for the completeness sake. GDMake uses the `GDMAKE_HOOK` with the address and the symbol parameter to hook a specific function and `GDMAKE_ORIG` keyword to call the original function. 

```cpp
GDMAKE_HOOK(0x1907b0, "_ZN9MenuLayer4initEv")
bool __fastcall MenuLayer_init(gd::MenuLayer* self, void* edx) {
    if (!GDMAKE_ORIG(self, edx)) return false;
    return true;
}
```

### Shared

Since all of these hooks are static functions, a `self` parameter and a parameter for clobbing the edx register is added (except MAT dash) to match the calling convention for member functions. These parameters need to be removed when moving the hook inside a modify class. Likewise, all uses of `self->` need to be either removed or replaced with `this->`.

Otherwise, all of the hooks can be replaced by a `Modify` class and the original calls can be replaced with a call the `OriginalClass::function` inside the modify hook with the needed parameters. 

```cpp
class $modify(MenuLayer) {
    bool init() {
        if (!MenuLayer::init()) return false;
        return true;
    }
}
```

## Patches

Patches are pretty easy to migrate. Geode has a `Mod::patch` function that takes in a byte vector and an address, which can be used for mod specific patches.

## Miscellaneous 

Some function signatures in gd.h are wrong: wrong as in they work for Windows, but not for any other platform. Geode uses the Android binary symbols to infer the function signatures, so the wrong function calls relating to this issue need to be fixed while migrating from gd.h.

## TODO

Add settings, data saving
