# Migrating from Geode v3.x to v4.0

## Changes to `Result`
[Link to the full docs](https://github.com/geode-sdk/result?tab=readme-ov-file#result)
* Result type has been rewritten from scratch, and it is now shared with many parts of the Geode codebase (Geode itself, TulipHook and matjson).
* Many methods removed or renamed:
    * `value()` -> `unwrap()`
    * `expect(str, fmt args...)` -> Use `mapErr` along with `fmt::format`
    * Snake case methods renamed to camel case
* Universal `GEODE_UNWRAP(res)` and `GEODE_UNWRAP_INTO(value, res)` macros
    * These macros were already previously available, but this is just a reminder that they can help a lot when dealing with results.
    ```cpp
    Result<std::string> addValues(matjson::Value json) {
       // Value::asInt() returns a Result<int>
       GEODE_UNWRAP_INTO(int a, json["a"].asInt());
       GEODE_UNWRAP_INTO(int b, json["b"].asInt());
       // must wrap return value with Ok
       return Ok(fmt::format("sum is {}", a + b));
    }
    ```
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
* Most SettingV3 code should work fine, with the exception of some Result usages:
```cpp
static Result<std::shared_ptr<MyCustomSettingV3>> parse(std::string const&, std::string const&, matjson::Value const& json) {
   auto res = std::make_shared<MyCustomSettingV3>();
   auto root = checkJson(json, "MyCustomSettingV3");
   // ... some code here ...
   return root.ok(res);
}
// should now be:
//                            ↓ here
static Result<std::shared_ptr<SettingV3>> parse(std::string const&, std::string const&, matjson::Value const& json) {
   auto res = std::make_shared<MyCustomSettingV3>();
   auto root = checkJson(json, "MyCustomSettingV3");
   // ... some code here ...
   //             ↓ here
   return root.ok(std::static_pointer_cast<SettingV3>(res));
}
```

## Changes to `utils::MiniFunction`
* Removed, use `std::function` now

## Changes to `geode::Layout`
* No longer in cocos2d namespace
    * Can now be found in `Geode/ui/Layout.hpp`

## Changes to `getChildOfType`
* Deprecated in 3.9.0, and now removed, use `CCNode::getChildByType<T>(int index)` instead:
* `getChildOfType<CCLayer>(node, 1)` -> `node->getChildByType<CCLayer>(1)`
* You can use this regex pattern to quickly find and replace:
   * `getChildOfType<(.+?)>\((.+?),\s*(.+?)\)` and replace with `$2->getChildByType<$1>($3)`

## Changes to `matjson`
[Link to the full docs](https://github.com/geode-sdk/json?tab=readme-ov-file#matjson)
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
       *  **Make sure to return a Result!** much of the code relies on `fromJson` returning a Result.
    * `matjson::Value to_json(T const& value)` -> `matjson::Value toJson(T const& value)`
    * `is_json` is no longer used, instead just return an error in `fromJson`
* `bool Value::is<T>()` removed
* `#include <matjson/stl_serialize.hpp>` -> `#include <matjson/std.hpp>`
* Added new experimental `<matjson/reflect.hpp>` header, uses [qlibs/reflect](https://github.com/qlibs/reflect) to (de)serialize aggregate structs
    * Behavior might change in the future, so use with caution

## Changes to `JsonChecker` / `JsonExpectedValue`
* The JsonChecker class was previously marked deprecated, and is now removed. Now you should move to using `JsonExpectedValue` instead
```cpp
auto checker = JsonChecker(json);
auto root = checker.root("[file.json]").obj();

// ... code

if (checker.isError()) {
    return Err(checker.getError());
}
```
is now
```cpp
auto root = checkJson(json, "[file.json]")

// ... code

GEODE_UNWRAP(root.ok());
```
* `JsonChecker::has` has different behavior to `JsonExpectedValue::has`, use `JsonExpectedValue::hasNullable` to keep old behavior with null values
