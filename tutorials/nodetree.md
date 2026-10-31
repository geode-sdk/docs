# Getting nodes

Finding nodes in the Cocos2d-x tree comes up in almost every mod, whether you want to read game state, change existing UI, or place your own content next to vanilla elements. Fragile lookups cause most mod incompatibility, crashes, and layout breakage after game updates or alongside other mods, so please pick the sturdiest method that fits.

In order of preference:

1. [Members](#getting-via-members)
2. [IDs via `getChildByID`](#getting-via-ids-using-getchildbyid)
3. [Type via `getChildByType`](#getting-via-type-using-getchildbytype)
4. [Queries via `querySelector`](#getting-via-queries-using-queryselector)
5. [Iteration via `CCArrayExt`](#getting-via-iteration-using-ccarrayext)
6. [More extreme approaches](#more-extreme-approaches)

Related: [Setting IDs](#setting-ids-using-setid), [Viewing string IDs](#viewing-string-ids), [If IDs are missing](#if-ids-are-missing).

Child order shifts with game logic and other mods, and the order DevTools shows after load often differs from the order during `init` where most mod code runs. So please treat index based lookup as a **last resort**.

## Getting via members

The most reliable way is **member variable access**. If the layer stores the node, for example `m_playtestBtn` on `EditorUI`, just use it:

```cpp
// inside $modify(EditorUI), no searching needed
m_playtestBtn->setVisible(false);
```

Inside `$modify`, your class inherits from the target, so you can use its members directly, including all `m_` fields from the bindings (which declare everything public, even members that are private in the original game code). Members stay stable across game updates and other mods. Only use the methods below when the node is **not** stored as a member.

## Getting via IDs using `getChildByID`

String IDs are a **Geode-specific addition** to `CCNode`. You can give any node a string ID and fetch a direct child by it. Unlike indexes, `node->getChildByID("specific-node-id")` keeps returning the same node:

```cpp
// inside MenuLayer::init
auto bottomMenu = this->getChildByID("bottom-menu");
auto settingsBtn = bottomMenu->getChildByID("settings-button");

auto name = static_cast<CCLabelBMFont*>(this->getChildByID("player-username"));
```

`getChildByID` only searches **immediate children** and returns `nullptr` when nothing matches, so please always check the result and only cast when you need a concrete type. `getChildByIDRecursive` searches the **whole subtree** instead, and `removeChildByID` is shorthand for `removeChild(getChildByID(...))`. Geode ships with [default IDs for vanilla layers](https://github.com/geode-sdk/geode/tree/main/loader/src/ids); for the rest, see [what to do if IDs are missing](#if-ids-are-missing).

## Setting IDs using `setID`

Every `CCNode` starts with an empty ID. Give it one with `setID`, and please do the same in your own layers to help debugging and other mods:

```cpp
auto label = Label::create("Hello!", "bigFont.fnt");
label->setID("hello-label");
m_mainLayer->addChild(label);
```

### IDs in your own layers: do not use `_spr`

Your own layer is fully yours, so use plain readable IDs like `"title-label"`. The `_spr` prefix only adds noise there.

### IDs added to game layers: always use `_spr`

Game layers are shared, so prefix IDs you add there with your mod ID to avoid collisions (`"author.mod-id/my-button"`). The `_spr` literal adds it for you:

```cpp
using namespace geode::prelude;

auto btn = Button::createWithSprite("my-cool-button.png"_spr, [](Button* sender) {});
btn->setID("my-cool-button"_spr);
menu->addChild(btn);
```

Vanilla IDs added by Geode stay plain (`getChildByID("main-menu")`). Use `_spr` when setting and getting your own nodes, and write out the full prefixed ID when using another mod's node (`getChildByID("geode.loader/geode-button")`).

### Naming string IDs

Use kebab-case with lowercase `a-z`, `0-9`, and `-`, without spaces: `"my-cool-button"`. The part after the `/` follows the same rule.

## Getting via type using `getChildByType`

When a node has no ID but you know its C++ type, use `CCNode::getChildByType`. It returns the matching child of that type at the given index, or `nullptr`:

```cpp
// bottom-menu is the 2nd CCMenu in MenuLayer
auto bottomMenu = this->getChildByType<CCMenu>(1);
```

The index defaults to `0` and negative values count back from the end. This pairs well with IDs: narrow down with `getChildByID`, then pick the unlabeled node inside by type.

## Getting via queries using `querySelector`

`CCNode::querySelector` finds nodes nested several levels deep with a small selector language. A space means "anywhere below" and `>` means "directly below":

```cpp
auto settingsBtn = this->querySelector("bottom-menu > settings-button");
auto userLabel = static_cast<CCLabelBMFont*>(this->querySelector("player-username"));
```

It returns the **first** match or `nullptr`, so please check the result. Only ID characters are allowed in queries. For simple one level lookups, prefer `getChildByID`.

## Getting via iteration using `CCArrayExt`

`CCArrayExt<T>` (from `<Geode/utils/cocos.hpp>`) makes it easier to loop through children. `getChildrenExt<T>()` wraps a node's children and `asExt<T>()` wraps any `CCArray`:

```cpp
#include <Geode/utils/cocos.hpp>

using namespace geode::prelude;

// Get all the buttons in the CCMenu
for (auto btn : bottomMenu->getChildrenExt<CCMenuItem>()) {}

// asExt works on any CCArray, not just children
for (auto obj : PlayLayer::get()->m_objects->asExt<GameObject>()) {}
```

It does not check types for you, so only use a derived type when every element has it. Otherwise iterate as the base type and filter with `typeinfo_cast`:

```cpp
for (auto child : this->getChildrenExt()) {
    if (auto btn = typeinfo_cast<CCMenuItemSpriteExtra*>(child)) {}
}
```

Use this to search by properties IDs cannot express (sprite, text, position) or to debug unlabeled children. If you repeat the same search in several mods, consider [assigning IDs once](#if-ids-are-missing) instead.

## More extreme approaches

> [!WARNING]
> These approaches are far more brittle, and we strongly encourage you to keep these approaches to a minimum if you publish a Geode mod using these functions for the sake of mod compatibility.

When nothing above fits, these helpers cover the stubborn cases. They all return `nullptr` when nothing matches, so please always check the result. A good rule of thumb is to use one just long enough to [give the node a stable ID](#if-ids-are-missing), then use that ID everywhere else.

### Sprite name via `getChildBySpriteName` and `getChildBySpriteFrameName`

If the only standout feature of a child is how it looks, search by its sprite (from `<Geode/utils/cocos.hpp>`):

```cpp
#include <Geode/utils/cocos.hpp>

// First immediate child using this image file
if (auto spr = geode::cocos::getChildBySpriteName(this, "GJ_button_01.png")) {}
// First immediate child using this sprite frame
if (auto spr = geode::cocos::getChildBySpriteFrameName(this, "GJ_playBtn_001.png")) {}
```

Use `getChildBySpriteName` for loose `.png` files and `getChildBySpriteFrameName` for sprites packed into a `.plist`. Both also find buttons showing that sprite, search **immediate children only**, and return the **first** match. For deeper searches, combine `isSpriteName` or `isSpriteFrameName` with a loop or `findFirstChildRecursive`. Sprite lookups stop matching when GD changes its sprites, so please turn a match into a `setID` right away.

### Tag via `getChildByTag` and `getChildByTagRecursive`

Tags are integers from Cocos2d-x. To find a direct child by tag:

```cpp
// Direct child with tag 42, or nullptr
if (auto found = this->getChildByTag(42)) {}
```

When the depth is unknown, search below a node instead (from `<Geode/utils/cocos.hpp>`):

```cpp
#include <Geode/utils/cocos.hpp>

// Tag 42 anywhere below this node, including this node
if (auto found = geode::cocos::getChildByTagRecursive(this, 42)) {}
```

Tags have no namespacing, so different mods can reuse the same tag. Please combine a tag match with a `typeinfo_cast` check when possible, and avoid tag `0`, which matches almost everything since it is the default.

### Index via `getChildByIndex`

Position based lookup breaks easily when another mod adds or moves a child. When you need it, please use the bounds checked lookup, which returns `nullptr` for a missing index. A negative index counts back from the end:

```cpp
// Child at position 5, or nullptr
if (auto menu = this->getChildByIndex<CCMenu>(5)) {}
auto lastChild = this->getChildByIndex(-1);
```

It searches **immediate children only** and does **not** verify the type, so please pass the correct type or `CCNode`. Only use an index when the position stays fixed, and avoid copying an index seen in DevTools after load into an `init` hook, since the order often differs there.

### Recommendation order

When nothing above applies, please try these in order:

1. `geode::cocos::getChildBySpriteName` for image files, then `geode::cocos::getChildBySpriteFrameName` for sprite frames.
2. `getChildByTag` or `geode::cocos::getChildByTagRecursive` when GD assigns a meaningful tag.
3. `getChildByIndex` when the position stays fixed and known.

## Viewing string IDs

The [DevTools](https://github.com/geode-sdk/DevTools) mod shows IDs in its **Tree** view.

## If IDs are missing

Geode ships default IDs for common layers, but many layers are still unlabeled. You can **add them yourself** (using members, `getChildByType`, or the [more extreme approaches](#more-extreme-approaches) above, ideally in `$modify::init` right after the original `init`):

```cpp
// Assign once so everyone can use getChildByID("bottom-menu")
this->getChildByType<CCMenu>(1)->setID("bottom-menu");
```

Then please [make a pull request to Node IDs](https://github.com/geode-sdk/nodeids) with your additions so every mod can use them. Its point is to **provide a single source of truth**; once someone has done the work, no one should have to do it again.
