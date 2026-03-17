# Geode Namespaces

This document describes the purpose of each namespace in the Geode SDK.

## Main Namespaces

### `geode`
The root namespace for all Geode functionality. Contains core classes and functions for mod development.

**Key classes:**
- `Loader` - Main entry point for interacting with Geode
- `Mod` - Represents a mod and its metadata
- `ModMetadata` - Information about a mod
- `Event`/`EventFilter` - Event system for mod communication

---

### `geode::modifier`
Contains utilities for modifying Geometry Dash classes and functions.

**Key classes:**
- `Modify` - Base class for creating hooks
- `Field` - For adding fields to existing classes
- `AsStaticFunction` - Converts member functions to static

**Usage:**
```cpp
using namespace geode::modifier;

class $modify(MyMenuLayer, MenuLayer) {
    // Your modifications here
};
```

---

### `geode::addresser`
Utilities for memory address manipulation and function resolution.

**Key functions:**
- `getNonVirtual` - Get address of non-virtual functions
- `getVirtual` - Get address of virtual functions
- `getThisAdjustment` - Get `this` pointer adjustments

---

### `geode::async`
Asynchronous programming utilities including coroutines and promises.

**Key classes:**
- `Promise<T>` - Represents a future value
- `Task<T>` - An async operation that can be started/cancelled
- `Coro` - Coroutine helper for async/await style code

**Usage:**
```cpp
geode::async::Task<std::string> fetchData() {
    auto result = co_await web::fetch("https://api.example.com");
    co_return result.string().unwrap();
}
```

---

### `geode::cocos`
Utilities for working with cocos2d-x (the game engine).

**Key classes:**
- `CCNodeRef` - Smart pointer for CCNode
- Various helper functions for node manipulation

---

### `geode::stl`
Standard library extensions and custom containers.

**Key classes:**
- `fixed_string` - String with compile-time size
- Custom STL allocators for Geode

---

### `geode::utils`
General utility functions.

**Sub-namespaces:**
- `geode::utils::file` - File operations
- `geode::utils::string` - String manipulation
- `geode::utils::web` - Web requests
- `geode::utils::cocos` - Cocos2d helper functions

---

## Platform Namespaces

### `geode::base`
Platform-specific base implementations. Usually not used directly.

### `geode::platform`
Platform detection and platform-specific code.

---

## Internal Namespaces

These namespaces are used internally by Geode and should generally not be used by mod developers:

- `geode::internal` - Internal implementation details
- `geode::geode_internal` - Core internal functions
- `geode::cast` - Internal casting utilities

---

## External Namespaces

### `gd`
Shortcut namespace for Geometry Dash classes. Contains bindings to GD classes.

**Usage:**
```cpp
// These are equivalent:
MenuLayer* layer;
gd::MenuLayer* layer;  // Shortcut
```

### `cocos2d`
The cocos2d-x game engine namespace. Contains all cocos2d classes like:
- `CCNode`, `CCLayer`, `CCScene` - Node hierarchy
- `CCSprite` - Images and textures
- `CCMenu`, `CCMenuItem` - UI elements
- `CCDirector` - Game management

---

## Best Practices

1. **Use `using namespace geode;`** in your `.cpp` files for convenience
2. **Use `using namespace geode::prelude;`** for the most common includes
3. **Don't use internal namespaces** - they're subject to change
4. **Prefer `gd::` shortcut** for GD class names
