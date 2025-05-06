# Image (Down)loading

If your have to display **images** in your mod that are **not included** in the mod file (for example for reducing the **size** of the mod or for showing **user-generated content**), you can use geode's `LazySprite` class. It can **also** be used for ordinary image initialization, either from a path or from raw data, which would have slight performance improvements over `CCSprite::create`, as the image is lazily read and decoded **in the background**, without freezing the game. 

It is **recommended** to use this API instead of implementing custom solutions that mess with `CCTextureCache`. It is not straightforward to do this correctly, and multiple mods (and Geode itself) have in the past failed to do this without memory leaks and other bugs.

By default, the sprite will have a **loading circle** inside it when it's loading, which is removed once loading is finished. This can be disabled, making the sprite blank until it's loaded.

## Usage

Basic examples of downloading an image from a URL, loading from a file, and initializing from data, and a showcase of various options.

```cpp
#include <Geode/ui/LazySprite.hpp>
using namespace geode::prelude;

// this will be the initial size of the node (and the loading circle)
CCSize nodeSize{64.f, 64.f};

auto spr1 = LazySprite::create(nodeSize);
// or, if you want to disable the loading circle
auto spr1 = LazySprite::create(nodeSize, false);

// if you want to know when the image finished loading, you can assign a callback
// do this **before** calling one of the next functions, because if the image is cached,
// the callback will be called instantly, potentially before you assign it.
spr1->setLoadCallback([spr1](Result<> res) {
    if (res) {
        log::info("Sprite loaded successfully!");
    } else {
        log::error("Sprite failed to load, setting fallback: {}", res.unwrapErr());
        spr1->initWithSpriteFrameName("globed-logo.png"_spr);
    }
});

// load the image from a URL
spr1->loadFromUrl("https://globed.dev/logo.png");
// or, load from a file
spr1->loadFromFile(Mod::get()->getSaveDir() / "file.png");
// or, load from binary data
std::vector<uint8_t> pngBytes = {0x1, 0x2, 0x3};
spr1->loadFromData(std::move(pngBytes));

// using cocos methods also works, but does not have any performance benefits:
spr1->initWithFile("my-file.png");
spr1->initWithSpriteFrameName("my-texture.png");

// note: you MUST either add the sprite as a child or store it in a Ref<>,
// if the sprite node gets deleted then it will stop loading and will never call your callback
someMenu->addChild(spr1);
```

## Automatic resize

Once the sprite has finished loading, the content size of the `LazySprite` will be set to match the loaded image (in the same manner a `CCSprite::initWithTexture` would). If you want the sprite to be automatically scaled to the size you passed in `LazySprite::create`, you can enable autoresize:

```cpp

auto spr1 = LazySprite::create({64.f, 64.f});
spr1->setAutoResize(true);
spr1->loadFromFile(Mod::get()->getSaveDir() / "file.png");

// now, once the sprite finished loading, it should be resized to 64x64 (in cocos units)
```

## Caching

By default, if using `loadFromFile` or `loadFromUrl`, this class will cache the textures in `CCTextureCache` (`loadFromData` is not supported due to it being hard to pick a uniquely identifying cache key).

If this is not wanted, the load functions have an extra bool argument for ignoring the cache:

```cpp
spr1->loadFromUrl("url", true);
```
