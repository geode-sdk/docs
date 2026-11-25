---
description: How to install dependencies to Geode mods
---

# Dependencies

Geode provides utilities for mods to depend on other mods, to make sharing code easy.

Note that Geode **only manages dependencies that are mods**. Normal C++ dependencies, like a JSON parsing library, ZIP library, or some networking utilities, **should just be installed like they are in any other C++ project**. Use CPM, Git submodules, copy the code to your project, whatever you prefer. If the library is dynamic, include the `.dll` with your mod through the `files` key in `resources`.

However, sometimes you want to depend on code that can't just be used as a normal library; for example, **custom keybinds**. Geode does not provide any custom keybinds utilities out-of-the-box, so you need to use a library. However, it would not make much sense if every mod bundled their own incompatible systems for dealing with custom keybinds. Instead, there should be one mod that just adds the functionality, and then other mods can depend on that mod and call its functions to add keybinds.

## How dependencies work

Dependency mods are like any other mods, except that they **include their headers and linkable files in their .geode package**. Usually the headers are located in an `include` directory inside the package, but they may also be anywhere. The linkable file is `mod.id.lib` on Windows and the normal mod binary on other platforms, located at the root of the package.

Otherwise, dependency mods are just like normal mods; they may place hooks, patches, add features to the game, and be published on the mods index. However, **dependency mods should keep the features they add to a minimum**, and be focused on the specific features they're meant to add. For example, a custom keybinds dependency should only add the necessities for working with custom keybinds; it shouldn't also add a bunch of other features, like adding more icons or customizing menus.

If a dependency is required, it is **linked to**; this means that for the mod that depends on it to run, the dependency must be present. However, as sometimes you may want to only use a dependency if it is loaded, **dependencies may also be marked optional**. In this case, the dependency is not linked to, which means that you can't use any of the dependency's functions, but have to use Geode's functions for working with optional dependencies. See [Optional dependencies](#Events) for more info.

## Adding dependencies

Dependencies can be added to your mod by simply adding it to the `dependencies` key in [mod.json](/mods/configuring):

```json
{
    "geode": "v1.0.0",
    "id": "my.example-mod",
    "name": "My example mod",
    "developer": "Me",
    "version": "v1.0.0",
    "dependencies": {
        "someones-mod": ">=v1.0.5",
        "someone-elses.mod": {
            "version": ">=v1.2.5",
            "required": true
        }
    }
}
```

Dependencies can be specified by using an object that maps from a mod id to a version, or more information if needed. The dependency may also be set as required, which specifies if the mod is required or [optional](#Optional-Dependencies) for the dependant mod to load. If this is not specified, the dependency is marked as required.

The `version` field of a dependency may be written as `>=version`, `=version`, or `<=version`. The comparisons work as expected, with the addition that if the major versions are different, the comparison is always false. This means that if you depend on version `>=1.2.5` of a mod, version `v1.8.0` will be considered acceptable but `v2.0.0` will not. For this reason, [if you make a mod that is depended upon, you should follow strict semver](https://semver.org/).

Once you have added a dependency to your `mod.json`, if you have [Geode CLI v1.4.0 or higher](/getting-started/geode-cli), its headers are automatically added to your project. If you have the mod installed, the headers from the installed version will be used. If you don't have the mod installed, Geode will install it from the mods index. The added dependency files for your project can be found in `build/geode-deps/<dep.id>`. Geode automatically adds `build/geode-deps` to your project's include path, and links whatever binaries are found in dependencies, meaning you most likely don't have to configure anything.

## Example

The mod `hjfod.gmd-api` contains utilities for working with [.GMD files](https://fileinfo.com/extension/gmd). You can add it to your mod by adding the following to your `mod.json`:

```json
{
    "dependencies": {
        "hjfod.gmd-api": ">=v1.0.0"
    }
}
```

Now, if you reconfigure your CMake (or if you're using CMake Tools in VS Code, it should be reconfigured automatically), Geode should automatically find `hjfod.gmd-api` in your installed mods. If the mods index, and install it for you.

```cpp
#include <hjfod.gmd-api/GMD.hpp>
// do things with gmd-api's functions
```

If you now go to compile and test your mod, everything should work out-of-the-box.

## Optional Dependencies

In case the dependency is **not required**, it is not linked to, which in turn means that you can't use any of its exported functions. In cases like these, the dependency should provide ways through Geode to dynamically call its functions, such as through events.

### Events

The key system Geode provides for optional mod interop are events. For example, a mod that adds support for drag-and-dropping files on the GD window could define a drag-and-drop event that other mods can then listen to.

See the [docs page on events](/tutorials/events) for more information.

### User Objects

One way for mods to communicate with optional dependencies is through **user objects**, which may contain any data. These are like the `setUserData` and `setUserObject` functions native to `CCNode`s, except that these objects have a string key associated with them. For example, a mod that adds scrollbars to layers might use the following check to see if a scrollbar should be added to a layer:

```cpp
if (layer->getUserObject("hjfod.cool-scrollbars/enable")) {
    // add scrollbar
}
```

Other mods can add this user object to their layers with the `CCNode::setUserObject` function:

```cpp
layer->setUserObject("hjfod.cool-scrollbars/enable", CCBool::create(true));
```

As these are typical `CCObject`s, a mod can store any piece of information within a user object. This can range from objects as simple as a `CCBool` to more complicated, mod-specific ones that store multiple pieces of data.

### User Flags

User flags are a frequently used alternative to user objects which does not store a value. This makes them a more optimized solution for our aforementioned scrollbar example.

```cpp
layer->setUserFlag("hjfod.cool-scrollbars/enable");
```

```cpp
if (layer->getUserFlag("hjfod.cool-scrollbars/enable")) {
    // add scrollbar
}
```

Do note that user flags and user objects are separate - `getUserFlag` and `getUserObject` are not interchangeable and need to be consistent with the setter used.

## Event Macro

Included in [Geode v4.3.0](https://github.com/geode-sdk/geode/releases/tag/v4.3.0) is a macro to automate the process of exporting functions from API mods, in a way that works for optional dependencies.

This makes use of events to grab a function pointer from the API mod, and call it from the dependent mod, hence the name.

### Example
```cpp
// (In your api distributed header file)
// (such as include/api.hpp)
#pragma once

#include <Geode/loader/EventExport.hpp>
// This undef is not strictly required, however the define that follows will throw a warning
// if MY_MOD_ID happens to be set by some other api and it is not undefined like this.
#undef MY_MOD_ID
// You must **manually** declare the mod id, as macros like GEODE_MOD_ID will not
// behave correctly to other mods using your api.
#define MY_MOD_ID "dev.my-api"

namespace api {
    // The canonical way for defining an exported function.
    // If the mod is not loaded or the function is unavailable, the result will be an error. Otherwise, it's the result returned by the API.
    GEODE_EVENT_EXPORT(Result<int>, factorial, (int x), (x));

    // If you prefer to *not* deal with results, that's an option too.
    // On failure, this will return a default value (0 in case of integers)
    GEODE_EVENT_EXPORT(int, factorialSimple, (int x), (x));

    // Overloads are natively supported:
    GEODE_EVENT_EXPORT(float, factorialSimple, (float x), (x));

    // With a different macro, you can even export member functions inside classes!
    struct Number {
        int m_value;

        // Same macro parameters, but with the class name at the start and `this` in the argument list
        GEODE_EVENT_EXPORT_MEMBER(Number, void, add, (const Number& other), (this, other));

        // Static members are supported with the `_STATIC` variant, syntax is the same as the non-member macro
        GEODE_EVENT_EXPORT_MEMBER_STATIC(int, random, (int min, int max), (min, max));

        // Const-qualified functions can be declared with the `_CONST` variant
        GEODE_EVENT_EXPORT_MEMBER_CONST(Number, Number, multiply, (const Number& other), (this, other));
    };
}
```

Then, in **one** of your source files, you must define the exported functions:

```cpp
// MUST be defined before including the header.
#define GEODE_DEFINE_EVENT_EXPORTS
#include "../include/api.hpp"

Result<int> api::addNumbers(int a, int b) {
    return Ok(a + b);
}

int api::factorialSimple(int x) {
    return 42;
}

float api::factorialSimple(float x) {
    return 42.f;
}

void api::Number::add(const Number& other) {
    m_value += other.m_value;
}

int api::Number::random(int min, int max) {
    return min + (max - min) / 2;
}

api::Number api::Number::multiply(const Number& other) const {
    return Number(m_value * other.m_value);
}
```

For the users of the API, calling is no different than calling regular C++ functions:
```cpp
#include <dev.my-api/api.hpp>
using namespace geode::prelude;

void test() {
    Result<int> r = api::factorial(4);
    if (r.isOk()) {
        log::info("Factorial: {}", r.unwrap());
    } else {
        log::warn("Failed to compute factorial: {}", r.unwrapErr());
    }

    int simple = api::factorialSimple(5);
    log::info("Factorial (or 0): {}", simple);

    int num = api::Number::random(10, 100);
    log::info("Random number: {}", num);
}
```