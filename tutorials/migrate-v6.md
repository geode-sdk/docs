# Migrating from Geode v5.x to v6.x

<!--toc:start-->
- [Migrating from Geode v5.x to v6.x](#migrating-from-geode-v5x-to-v6x)
  - [geode::Label](#geodelabel)
  - [geode::Popup](#geodepopup)
  - [Tasks](#tasks)
  - [Keybinds](#keybinds)
  - [Settings](#settings)
  - [Miscellaneous](#miscellaneous)
<!--toc:end-->

> :warning: Geode v6.x is still *work-in-progress*, this guide will be updated
> as breaking changes are made

## geode::Label

`geode::Label` is a new (also available in **Geode v5**), highly
efficient alternative to `CCLabelBMFont`. It is **NOT** a subclass of
`CCLabelBMFont`, but it does implement `CCLabelProtocol` and `CCRGBAProtocol`.
Additionally, `geode::Label` also implements `CCBlendProtocol`.
Besides the new API, the Label also works a little differently under the hood.
Instead of containing one `CCSprite` child for each glyph, the Label renders text
using **OpenGL quads**.

Compared to `CCLabelBMFont`, `geode::Label` brings much improved performance,
**Unicode glyph support** (if the font you're using contains them), **emojis**
and its own separate **font cache**.

You are *not* required to use `geode::Label` in your mods, but all Geode UI
elements have been migrated to it.

Here's a small example:

```cpp
// ------------
// -- BEFORE --
// ------------
cocos2d::CCLabelBMFont* label = cocos2d::CCLabelBMFont::create(
  "Hello, world!", "bigFont.fnt"
);

label->setString("new string");

std::string variable = "string from variable";

// setString requires a const char*
label->setString(variable.c_str());
// max width, max scale, min scale
label->limitLabelWidth(180.0f, 0.6f, 0.1f);
label->setFntFile("goldFont.fnt");
const char* fontFile = label->getFntFile();
CCBMFontConfiguration* font = label->getConfiguration();

label->setAlignment(kCCTextAlignmentCenter); // ugly!

// -----------
// -- AFTER --
// -----------
#include <Geode/ui/Label.hpp>

geode::Label* label = geode::Label::create("Hello, world!", "bigFont.fnt");

label->setString("new string");

std::string variable = "string from variable";

// setString requires a const char*
label->setString(variable.c_str());
// setText requires a `std::string`, so...
label->setText(variable);
// we can move into it, taking ownership of the variable
label->setText(std::move(variable));
label->setRichText("We can also set <cr>colored</c> text!");

// instead of `limitLabelWidth`, we use `setLimitLabelWidth`, and...
label->setLimitLabelWidth(180.0f, 0.6f, 0.1f);
// we can also...
label->setLimitLabelHeight(32.0f, 0.6f, 0.1f);
// do more!
label->setLimitLabelSize(cocos2d::CCSize{ 180.0f, 32.0f }, 0.6f, 0.1f);
// or don't...
label->removeLabelSizeLimit();

label->setExtraKerning(0.5f); // we can even set some kerning

label->setMaxWidth(150.0f); // or configure wrapping
label->setBreakWords(true);

label->setFont("goldFont.fnt"); // can also take a `BitmapFont*`
ZStringView fontFile = label->getFontFile();
BitmapFont* font = label->getFont();

label->setAlignment(geode::Label::Alignment::Center); // beautiful!
```

As a quick migration cheat sheet:

| cocos2d::CCLabelBMFont                              | geode::Label                                           |
|-----------------------------------------------------|--------------------------------------------------------|
| `cocos2d::CCLabelBMFont::create(str, fnt)`          | `geode::Label::create(str, fnt)`                       |
| `label->setWidth(width)`                            | `label->setMaxWidth(width)`                            |
| `label->setAlignment(kCCTextAlignmentCenter)`       | `label->setAlignment(geode::Label::Alignment::Center)` |
| `label->limitLabelWidth(width, maxscale, minscale)` | `label->setLimitLabelWidth(width, maxscale, minscale)` |
| `label->m_nExtraKerning = x`                        | `label->setExtraKerning(x)`                            |
| `label->setFntFile(file)`                           | `label->setFont(file)`                                 |
| `label->getConfiguration()`                         | `label->getFont()`                                     |

As a result of the migration to `geode::Label`, the following breaking changes were made:

- `SettingNodeV3::getNameLabel()`: changed return type
- `SettingNodeV3::getStatusLabel()`: changed return type
- `IconButtonSprite::getLabel()`: changed return type
- `Notification::getLabel()`: changed return type
- `Popup::m_title`: is now `geode::Label`
- `ProgressBar::getProgressLabel()`: changed return type
- `SelectList::m_label`: is now `geode::Label`
- `SliderNode::linkLabel()`: requires `geode::Label` for 1st param
- `SliderNode::getLinkedLabel()`: changed return type
- `SimpleTextArea::getLines()`: returns `std::vector<geode::Label*>`

## geode::Popup

- `Popup::m_title` is now of type `geode::Label`
- The vanilla **fast menu** option will now be respected by default

## Tasks

- Tasks have been **completely removed**. Please switch to using [async](/tutorials/async)
for all your task-related needs

## Keybinds

- Added keybind modifier **fallthrough** (see [this pull request](https://github.com/geode-sdk/geode/pull/2000) for more information)

## Settings

- `SettingV3::updateState2()` has been removed
- `SettingV3::updateState()` now has **public** visibility

## Miscellaneous

- (ABI break) `geode::utils::game::restart(bool saveData)` has been removed; you
will have to use `geode::utils::game::restart(bool saveData, bool safeMode = false)`
- Updated **fmtlib** to **12.2.0**
