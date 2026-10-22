# Migrating from Geode v5.x to v6.x

<!--toc:start-->
- [Migrating from Geode v5.x to v6.x](#migrating-from-geode-v5x-to-v6x)
  - [Filesystem](#filesystem)
  - [Event exports](#event-exports)
  - [geode::Label](#geodelabel)
  - [geode::Popup](#geodepopup)
  - [Tasks](#tasks)
  - [Keybinds](#keybinds)
  - [Settings](#settings)
  - [GD and cocos2d headers](#gd-and-cocos2d-headers)
  - [Miscellaneous](#miscellaneous)
<!--toc:end-->

> :warning: Geode v6.x is still *work-in-progress*, this guide will be updated
> as breaking changes are made

## Filesystem

Due to issues with encoding and an easy-to-misuse exception based API, Geode v6 fully abandons the `std::filesystem` module, with the `asp::fs` module of the `asp` library serving as a replacement. You are *not* required to use it everywhere, but all Geode APIs have been switched to it, so functions like `Mod::getSaveDir` will all return a `asp::fs::Path`.

The `asp::fs::Path` and `asp::fs::PathView` classes are essentially wrappers over a `std::string` and `std::string_view`, but with a bunch of path/filesystem related functions, so replacing `std::filesystem::path` is easy.

> :warning: One of the biggest differences in the API is that `extension()` returns extension without the dot, aka `txt`, `exe`. This is different from `std::filesystem` which includes the dot. This is a very easy thing to miss, as your compiler won't warn you but your checks might work incorrectly.

```cpp
// old, std::filesystem based code
std::filesystem::path dir = Mod::get()->getSaveDir();

std::error_code ec;
if (!std::filesystem::exists(dir, ec)) {
    return;
}

std::filesystem::remove_all(dir, ec);
std::filesystem::create_directories(dir, ec);

auto filePath = dir / "file.txt";
std::ofstream stream(filePath);
stream << "hi" << std::endl;
stream.close();

for (auto entry : std::filesystem::directory_iterator(filePath.parent_path())) {
    if (entry.is_regular_file()) {
        log::debug("file: {}", utils::string::pathToString(entry.path()));
    }
}


// new, asp::fs based code
asp::fs::Path dir = Mod::get()->getSaveDir();
if (!dir.exists()) {
}

// asp::fs functions cannot throw, but they return a Result that you should handle
auto result = asp::fs::removeDirAll(dir);
if (!result) {
    log::warn("failed to delete: {}", result.unwrapErr());
    return;
}

// for the sake of keeping the example simple, we will just ignore the error string in next lines
if (asp::fs::createDirAll(dir).isErr()) return;

auto filePath = dir / "file.txt";
auto file = asp::fs::File::create(dir / "file.txt").unwrap();
if (!file.write("hi\n")) return;

file.close(); // optional, to flush immediately

auto iterator = asp::fs::iterDir(filePath.parent().value());
if (!iterator) return;

for (auto result : iterator.unwrap()) {
    if (!result) return;
    auto& entry = result.unwrap();
    
    if (entry.isFile()) {
        log::debug("file: {}", entry.path.string()); // no more pathToString!
    }
}
```

If this seems like too much result handling, we recommend converting your functions that perform many file operations to also be Result-based:
```cpp
Result<> processDir(asp::fs::Path dir) {
    GEODE_UNWRAP_INTO(auto iterator, asp::fs::iterDir(dir));
    for (auto result : iterator) {
        GEODE_UNWRAP_INTO(auto entry, result);

        if (entry.isFile()) {
            // ...
        }
    }

    return Ok();
}

// then, the function returns an error if anything failed:
auto result = processDir("files");
if (!result) {
    log::warn("processing failed: {}", result.unwrapErr());
}
```

There are multiple benefits to using `asp::fs` APIs:
* Less accidental encoding issues; paths returned by system APIs are (typically on Unix, guaranteed on Windows) **UTF-8**, instead of using system codepage like `std::fs::path::string()`.
* Same and consistent path everywhere, no wide strings
* No hidden exceptions, convenient `Result` based API you know and love
* Small and contained headers that are much lighter on compile time than heavy `<filesystem>` includes

However, if this doesn't suit you and you have to call some `std::filesystem` functions, you can always convert paths back and forth by including `<asp/fs/std.hpp>` and calling `Path::fromStdPath` or `toStdPath()`.

## Event exports

The syntax of the `GEODE_EVENT_EXPORT` macro was changed completely, due to previous syntax not being flexible enough. The new syntax fixes ODR violations (it is now legal to call exported functions in the same mod that exports them) and overloads work out of the box, without needing to define IDs:

```cpp
// Old
inline Result<int> addNumbers(int a, int b) GEODE_EVENT_EXPORT(&addNumbers, (a, b));
inline Result<int> addNumbers2(float a, float b) GEODE_EVENT_EXPORT(&addNumbers2, (a, b));
// ^ overload would error unless we use another name or define event ID manually

struct Test {
    Result<int> addNumbers(int a, int b) GEODE_EVENT_EXPORT(&Test::addNumbers, (this, a, b));
};


// New
// macro arguments: (return, name, args, call)
GEODE_EVENT_EXPORT(Result<int>, addNumbers, (int a, int b), (a, b));
GEODE_EVENT_EXPORT(Result<int>, addNumbers, (float a, float b), (a, b));

struct Test {
    // note the `_MEMBER` in the name and class name as first arg
    GEODE_EVENT_EXPORT_MEMBER(Test, Result<int>, addNumbers, (float a, float b), (this, a, b));
};
```

Another change is that the `_NORES` versions of the macro are gone, and the return value on error is determined automatically:
```cpp
// If the mod is not loaded, calling `addNumbers` with this definition will return error "Unable to call method"
GEODE_EVENT_EXPORT(Result<int>, addNumbers, (int a, int b), (a, b));

// And with this signature, it will simply return 0, like the GEODE_EVENT_EXPORT_NORES macro used to
GEODE_EVENT_EXPORT(int, addNumbers, (int a, int b), (a, b));
```

Other aspects are unchanged: the `.cpp` file with function definitions stay the same, and consumers call the functions as they did before. Only the API headers need to change.

## geode::Label

`geode::Label` is a new (also available in **Geode v5**), highly
efficient alternative to `CCLabelBMFont`. It is **NOT** a subclass of
`CCLabelBMFont`, but it does implement `CCLabelProtocol` and `CCRGBAProtocol`.
Additionally, `geode::Label` also implements `CCBlendProtocol`.
Besides the new API, the Label also works a little differently under the hood.
Instead of containing one `CCSprite` child for each glyph, the Label renders text
using **quads**.

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
- `Popup::onClose` is now public
- The vanilla **fast menu** option will now be respected by default

## Tasks

- Legacy tasks (`geode::Task`) have been **completely removed**. Please switch to using [async](/tutorials/async)
for all your task-related needs. This does *not* concern `arc::Task`.

## Keybinds

- Added keybind modifier **fallthrough** (see [this pull request](https://github.com/geode-sdk/geode/pull/2000) for more information)
- No solution has been implemented to integrate with RobTop's keybind system *as of now*

## Settings

- `SettingNodeV3::updateState2()` has been removed
- `SettingNodeV3::updateState()` now has **public** visibility

## GD and cocos2d headers

- Replaced all usage of `gd::*` classes with `std::*`, bye bye gnustl!
- `CCMouseDelegate::scrollWheel` returns `bool`
- `CCTouchDispatcher::registerForcePrio` takes an additional `int`
- `ZipUtils::compressString` gains a `level` argument and `ZipUtils::ccDeflateMemory` gains an `int` argument
- `DS_Dictionary::loadRootSubDictToCompressedFile`, `DS_Dictionary::saveRootSubDictToCompressedFile` gained an additional `bool` argument
- Updated `CCKeyboardDispatcher` member names
- Renamed `CCNode::m_bUnkBool2` to `CCNode::m_bDisableScheduling`

## Miscellaneous

- (ABI break) `geode::utils::game::restart(bool saveData)` has been removed; you
will have to use `geode::utils::game::restart(bool saveData, bool safeMode = false)`
- Updated **fmtlib** to **12.2.0**
- `geode::utils::cocos::switchToScene` now has a `replace` argument. It defaults to `false` making the default behavior `pushScene` instead of the former `replaceScene`.
- Native binaries path has changed from `resources/<modid>/binaries` to just `binaries`
- Reduced includes across the whole codebase, you might need to import some additional headers
- Removed `utils::Timer`, `LogPerformance`
- Moved sha256 utilities from `Geode/utils/hash.hpp` into `Geode/utils/crypto.hpp`
- `async::waitForMainThread` now infers return type instead of returning `void`
