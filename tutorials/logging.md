# Logging

Geode provides a builtin logger with [fmtlib](https://fmt.dev/latest/index.html) formatting, which you can view in real time via the [platform console](#platform-console), or later on by checking the logs in the `geode/logs/` folder.

## Log Levels
There are 4 logging levels, each being more important than the previous.
```cpp
using namespace geode;

log::debug("Im some debug string {}", 123);
log::info("Some information for those reading logs");
log::warn("Warning: something idk");
log::error("An error message");
```

## Syntax
The syntax for formatting logs is the same as [fmtlib](https://fmt.dev/latest/syntax.html). Geode itself provides formatters for a few common types, such as `CCPoint`, `CCNode*`, etc.

![Cheatsheet showing fmtlib syntax](https://hackingcpp.com/cpp/libs/fmt.png)
*Image from [https://hackingcpp.com](https://hackingcpp.com/cpp/cheat_sheets.html)*

As such, you are able to do things like this:
```cpp
geode::log::debug("Hello {:>10.3f} world, [{}]", 12.4f, fmt::join(someVec, ", "));
```

## Platform console
You can easily enable the platform console by going to the settings on the Geode mod ingame.

![Image showing the platform console option ingame](/assets/geode_platform_console.png)
