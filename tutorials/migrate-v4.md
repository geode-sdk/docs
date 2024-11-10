# Migrating from Geode v3.x to v4.0

## Changes to `Result`
* Result type has been rewritten from scratch, and it is now shared with many parts of the Geode codebase (Geode itself, TulipHook and matjson).
* Many methods removed or renamed:
    * `value()` -> `unwrap()`
    * `expect(str, fmt args...)` -> Use `mapErr` along with `fmt::format`
    * Snake case methods renamed to camel case
* Universal `GEODE_UNWRAP(res)` and `GEODE_UNWRAP_INTO(value, res)` macros
    * These macros were already previously available, but this is just a reminder that they can help a lot when dealing with results.
    * If you only ever expect to build your code with **clang**, you can use the `GEODE_UNWRAP` macro like so, for convenience:
    ```cpp
    Result<int> getInt();

    Result<int> foo(int extra) {
        // This will not compile on MSVC
        int value = GEODE_UNWRAP(getInt()) + extra;
        return Ok(value);
    }
    ```

## Changes to the Geode Settings API
* Settings v2 has been removed, V3 suffix from classes are now aliased
* Code that already used SettingsV3 should still work

## Changes to `geode::Layout`
* No longer in cocos2d namespace
    * Can now be found in `Geode/ui/Layout.hpp`

## Changes to `matjson`
[Link to the full docs](https://github.com/geode-sdk/json)
* Entire library rewritten to use `geode::Result`
    * See result section for tips on how to use Geode's Result class
* Methods are now camel case to fit with rest of Geode's codebase
    * `is_string` -> `isString`
    * `as_string` -> `asString`
    * etc..
* `matjson::Object` has been removed, now you can just iterate directly off a Value (same for arrays!)
    ```cpp
    matjson::Value someObject = ...;
    for (auto& [key, value] : someObject) {
        log::debug("{}: {}", key, value);
    }

    matjson::Value someArray = ...;
    for (auto& value : someArray) {
        log::debug("{}", value);
    }
    ```
* Use `matjson::makeObject` for making objects inline now:
    ```cpp
    matjson::Value obj = matjson::makeObject({
        { "key", 123 },
        { "another-key": "another value!" }
    });
    ```
* Accessing missing properties with `Value::operator[]` will now return a null value instead of throwing an exception
* Serialization methods have been changed:
    * `T from_json(matjson::Value const&)` -> `Result<T> fromJson(matjson::Value const&)`
    * `matjson::Value to_json(T const& value)` -> `matjson::Value toJson(T const& value)`
    * `is_json` is no longer used, instead just return an error in `fromJson`
* `bool Value::is<T>()` removed
* `#include <matjson/stl_serialize.hpp>` -> `#include <matjson/std.hpp>`
* Added new experimental `<matjson/reflect.hpp>` header, uses [qlibs/reflect](https://github.com/qlibs/reflect) to (de)serialize aggregate structs
    * Behavior might change in the future, so use with caution