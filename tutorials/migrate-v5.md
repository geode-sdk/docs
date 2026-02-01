# Migrating from Geode v4.x to v5.0

## C++ Standard

Geode now requires C++23 to build. The mod template has been updated a while ago, but for older mods you might need to open `CMakeLists.txt` and edit this line:
```cmake
set(CMAKE_CXX_STANDARD 20)
```

to say `23` instead of `20`.

## Changes to `Popup`

`geode::Popup` is no longer templated, and instead accepts its own arguments in `Popup::init` (which also replaces `initAnchored`). It now uses the pattern similar to any other node. For example the following code:

```cpp
class MyPopup : public Popup<int> {
public:
    static MyPopup* create(int value) {
        auto popup = new MyPopup;
        if (popup->initAnchored(320.f, 160.f, value)) {
            popup->autorelease();
            return ret;
        }
        delete ret;
        return nullptr;
    }

protected:
    bool setup(int value) override {
        auto node = CCLabelBMFont::create("Hi", "bigFont.fnt");
        this->addChild(node);

        return true;
    }
};
```

Should be refactored to this:

```cpp
class MyPopup : public Popup {
public:
    static MyPopup* create(int value) {
        auto popup = new MyPopup;
        if (popup->init(value)) {
            popup->autorelease();
            return ret;
        }
        delete ret;
        return nullptr;
    }

protected:
    bool init(int value) {
        if (!Popup::init(320.f, 160.f))
            return false;

        auto node = CCLabelBMFont::create("Hi", "bigFont.fnt");
        this->addChild(node);

        return true;
    }
};
```

## Async changes (web, file picker, etc.)

`geode::Task` has been replaced by an asynchronous system that utilizes C++ coroutines. This makes it much easier to write performant, concurrent code, but it also means there's a lot to change to port to the new system.

Functions like `geode::utils::file::pick` or `web::WebRequest::get` now return a Future that can be awaited or spawned to run in parallel. For example, to launch a global file picker in v4, you would use this code:

```cpp
file::pick(...).listen([](Result<std::filesystem::path>* path) {
    if (path && path->isOk()) {
        auto path = path->unwrap();
    }
});
```

The new version for that would be:
```cpp
#include <Geode/utils/async.hpp>

async::spawn(
    file::pick(...),
    [](Result<std::optional<std::filesystem::path>> result) { // note that this is not a pointer anymore!
        if (result.isOk()) {
            auto opt = path.unwrap();
            if (opt) {
                auto path = opt.value();
            } else {
                // User cancelled the dialog
            }
        }
    }
);
```

This will behave similar to original, it will spawn `file::pick()` on the async runtime, and once completed, call the provided callback on the main thread. If you want more control over what happens (such as not calling your function on main thread), you can use the `Runtime` directly:
```cpp
arc::Runtime& rt = async::runtime();
auto handle = rt.spawn([](this auto self) -> arc::Future<> {
    auto result = co_await file::pick(...);
    if (result.isOk()) {
        auto path = result.unwrap();
    }
}());
```

If you've instead used listeners, a `async::TaskHolder` class was added which has a similar API. For example, the following v4 code that makes requests:

```cpp
EventListener<WebTask> listener;

auto req = WebRequest().get("https://example.org");
listener.bind([](WebTask::Event* e) {
    if (WebResponse* value = e->getValue()) {
        log::debug("Response: ", value->code());
    } else if (WebProgress* progress = e->getProgress()) {
        log::debug("Progress: ", progress->downloadProgress());
    }
});
listener.setFilter(req);
```

should now be changed to this:

```cpp
async::TaskHolder<WebResponse> listener;

auto req = WebRequest();

// optional progress callback, you can also directly poll via `req.progress()`
req.onProgress([](WebProgress const& progress) {
    log::debug("Progress: ", progress.downloadProgress());
});

listener.spawn(
    req.get("https://example.org"),
    [](WebResponse value) {
        log::debug("Response: {}", value.code());
    }
);
```

## Async cancellation

The `geode::Task` class allowed tasks to be cancelled externally, or even by the task itself. The new futures are cancelled simply by destroying them, and tasks are cancelled by calling `abort()` on the handle. The aforementioned `TaskHolder` class automatically will call `abort` on the held task when destroyed, replaced by another `.spawn` call, or when you manually call `listener.cancel()`. For example, this code below will never actually complete the request:
```cpp
async::TaskHolder<WebResponse> listener;

listener.spawn(
    WebRequest().get("https://example.org"),
    [](auto value) {}
);
listener.cancel();
```

When rewriting your own code that used `Task`, you can forget about checking for cancellation and leverage the power of coroutines - your task can be cancelled at any `co_await` point.

## `fmt::localtime`

We updated our `fmt` dependency to v12, which removed the `fmt::localtime` function. Now Geode provides a drop-in replacement for it:
```cpp
#include <Geode/utils/general.hpp>

std::time_t time = std::time(nullptr);
std::tm local = geode::localtime(time);
```

## Changes to `std::function` arguments

Almost all parts of geode that accepted a `std::function` (such as `queueInMainThread`, `TextInput::setCallback`, etc.) have been changed to use `geode::Function` instead. The primary difference is that a `geode::Function` cannot be copied, only moved. Most proper usages should still compile and work properly, but if you have errors that are related to `std::function`, try adding `std::move`s or making sure to explicitly use `geode::Function` everywhere.

## Changes to string arguments

A ton of functions that accepted `std::string_view`, `std::string` or `std::string const&` have been written uncarefully and inefficiently before, which v5 aimed to fix. For many cases, you don't need to change anything, but conversion between string types isn't always possible, so this might break some code. This applies to return values as well, certain commonly used functions like `CCNode::getID` have been changed to return `geode::ZStringView`, which is a type that attempts to be API-compatible with `string_view`, while keeping a guarantee of null termination. Here are some examples of code that breaks and how to fix it:

TODO