# Migrating from Geode v4.x to v5.0

## C++ Standard

Geode now requires C++23 to build. The mod template has been updated a while ago, but for older mods you might need to open `CMakeLists.txt` and edit this line:
```cmake
set(CMAKE_CXX_STANDARD 20)
```

to say `23` instead of `20`.

## Changes to `Event`

The entire event system has been reworked. There are now multiple event classes for different purposes, and filters are far more performant, though a lot less dynamic. \
Events now also have priority ordering, for defining the order in which listeners are handled.

### Usage migration

```cpp
// BEFORE:
// # Sending an event
PlayerHealthEvent("joe123", 65).post();

// # Listening to an event
// leaking the EventListener
new EventListener<EventFilter<PlayerHealthEvent>>(+[](PlayerHealthEvent* ev) {
    if (ev->getName() == "alice_") {
        log::info("Alice's health is now {}", ev->getHealth());
    }
    return ListenerResult::Propagate;
});
```

```cpp
// AFTER:
// # Sending an event
PlayerHealthEvent("joe123").send(65);

// # Listening to an event
// listener is a geode::comm::ListenerHandle
auto listener = PlayerHealthEvent("alice_").listen([](int health) {
    log::info("Alice's health is now {}", health);
    return ListenerResult::Propagate;
    // return false; would also be equivalent
});
// destroying the listener will prevent our callback from being called,
// so leaking it is an option
listener.leak();

// # Priorities
PlayerHealthEvent("joe123").listen([](int health) {
    // if the lambda returns void then its treated the same as propagate
}, Priority::VeryEarly);
```

### Migration for Event subclasses

```cpp
// BEFORE:
class PlayerHealthEvent : public Event {
public:
    PlayerHealthEvent(std::string name, int health);

    std::string m_name;
    int m_health;
};
```

```cpp
// AFTER: the std::string is a filter argument for the player's name
class PlayerHealthEvent : public Event<PlayerHealthEvent, bool(int health), std::string> {
public:
    using Event::Event;
};
```

### Migration for `DispatchEvent`

Renamed to `geode::Dispatch`, but `DispatchEvent` is kept as an alias for easier migration. Its a thread-safe event with one filter arg for the id:

```cpp
template <class... Args>
class Dispatch : public ThreadSafeEvent<Dispatch<Args...>, bool(Args...), std::string> {
    // [...]
};

template<class... Args>
using DispatchEvent = Dispatch<Args...>;
```

Usage is the same as a regular event usage

### Thread safety

Thread safety is now opt-in for events, meaning that if you want to send/listen to an event from different threads you should use the `ThreadSafe` variants.

```
Event -> ThreadSafeEvent
GlobalEvent -> ThreadSafeGlobalEvent
```

Note that this has nothing to do with GD's main thread, event listeners will always get triggered on the same thread that sent the event.

### `GlobalEvent`

If you have an `Event` with filter args, it's not possible to have an unfiltered listener (aka a listener that receives every event, regardless of the args). \
In those cases, you should use `GlobalEvent`.

```cpp
// std::string is a filter arg
struct MyEvent : GlobalEvent<MyEvent, bool(int value), std::string> {
    using GlobalEvent::GlobalEvent;
};
// if you want to be efficient and not copy the std::string for each listener you can use:
// struct MyEvent : GlobalEvent<MyEvent, bool(std::string_view id, int value), bool(int value), std::string> { [...] }
```
Usage is mostly the same, except for `listen`:
```cpp
// These are both valid:
MyEvent().listen([](auto id, auto value) {
    // will receive all events, unfiltered
});

MyEvent("some-id").listen([](auto value) {
    // will only trigger when id is "some-id"
});
```

## Changes to `Popup`

`geode::Popup` is no longer templated, and instead accepts its own arguments in `Popup::init` (which also replaces `initAnchored`). It now uses the pattern similar to any other node. For example the following code:

```cpp
// BEFORE:
class MyPopup : public Popup<int> {
public:
    static MyPopup* create(int value) {
        auto popup = new MyPopup;
        if (popup->initAnchored(320.f, 160.f, value)) {
            popup->autorelease();
            return popup;
        }
        delete popup;
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
            return popup;
        }
        delete popup;
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
            auto opt = result.unwrap();
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
auto handle = rt.spawn([] -> arc::Future<> {
    auto result = co_await file::pick(...);
    if (result.isOk()) {
        auto path = result.unwrap();
    }
});
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

// optional progress callback, you can also directly poll via `req.getProgress()`
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

## Async in general

See the [Async](/tutorials/async) page for in-depth overview of async.

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

## Changes to dependencies and incompatibilities

Dependencies and incompatibilites have received some breaking changes. The first is that the old array syntax has now been fully removed, and the second is that the old `"importance"` keys have been replaced by `"required": boolean` for dependencies and `"breaking": boolean` for incompatibilities. Suggestions and recommendations have been removed, but will be added back in a later update as a separate key.

For example, the following v4 code:

```json
{
    "dependencies": [
        {
            "id": "hjfod.gmd-api",
            "version": "1.2.1",
            "importance": "required"
        },
        {
            "id": "hjfod.trashcan",
            "version": "1.0.0",
            "importance": "recommended"
        },
        {
            "id": "alphalaneous.awesome_modifier_icons",
            "version": "1.0.3",
            "importance": "suggested"
        }
    ],
    "incompatibilities": [
        {
            "id": "alphalaneous.improved_group_view",
            "version": "1.0.0",
            "importance": "breaking"
        }
    ]
}
```

should now be written as:

```json
{
    "dependencies": {
        // Required dependencies can be written with the shorthand syntax
        "hjfod.gmd-api": "1.2.1",
        "hjfod.trashcan": {
            "version": "1.0.0",
            "required": false
        }
    },
    "incompatibilities": {
        // Or with the long form { "breaking": true, "version": "1.0.0" }
        "alphalaneous.improved_group_view": "1.0.0"
    }
}
```

## Changes to mod error reporting

 * `Mod::isEnabled` has been renamed to `Mod::isLoaded` to better describe its purpose
 * `Mod::isOrWillBeEnabled` now reports whether the user has tried to enable the mod and does not consider if it succesfully loaded or not
 * The `LoadProblem` class now has significantly fewer variants.
   * All of the different functions related to checking different problems in `Mod` have been reduced to just `Mod::failedToLoad` (for major issues) and `Mod::getLoadProblem` (for all potential issues)
 * `ModMetadata` parsing has been reduced to just the `create` and `createFromGeodeFile` functions, which now always return a `ModMetadata` instead of a result. The returned `ModMetadata` represents a best-effort parse, and any errors are reported in the `ModMetadata` itself that you can check via `hasErrors()` and `getErrors()`.

## getChild deprecation
`getChild` in the global namespace has been removed, use CCNode::getChildByIndex instead, the behavior is identical.

## "path" setting removal
`path` setting type has been removed, use `file` instead, the behavior is identical.
