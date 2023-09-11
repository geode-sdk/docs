# Utils

Along with Geode's many features and luxuries that make modding the game itself much easier, Geode also comes with tons of utilities to make some basic tasks alot easier to do in a Geode mod. These should **always** be preferred against any other method that could be platform specific.

> :info: This list does not cover absolutely all of the utilities that Geode has to offer. You can see all of them in [the repository](https://github.com/geode-sdk/geode/tree/main/loader/include/Geode/utils).

## General Utilities

### `strfmt`

Formats a `std::string`

### `cstrfmt`

Formats a `char const*`

### `intToHex`

Simply converts a number to hex in `std::string`

### `numToString`

Turn a number into a string, with support for specifying precision (unlike `std::to_string`).

### `timePointToString`

Converts a `std::chrono::system_clock::time_point` to a `std::string`

## File Utilities

### `readString`

### `readJson`

### `readBinary`

### `createDirectory`

### `readDirectory`

## Cocos Utilities

### `calculateNodeCoverage`

Get bounds for a set of nodes. Based on content size.

### `switchToScene`

Create a CCScene from a layer and switch to it with the default fade transition.

### `reloadTextures`

Reload textures, overwriting the scene to return to after the loading screen is finished.

### `limitNodeSize`

Rescale node to fit inside given size.

### `nodeIsVisible`

Checks if a node is visible (recursively checks parent visibility).

### `getChildByTagRecursive`

Gets a node by tag by traversing children recursively.

### `findFirstChildRecursive`

Get first node that conforms to the predicate by traversing children recursively.

> :warning: Both `getChildByTagRecursive` and `findFirstChildRecursive` should **never** be the primary way to find nodes. Always prioritize using Geode's [string ID system](/tutorials/nodetree.md#string-ids) to find nodes.

### `fileExistsInSearchPaths`

Checks if a given file exists in CCFileUtils search paths.

### `invert4B`

Inverts a `ccColor4B` color.

### `invert3B`

Inverts a `ccColor3B` color.

### `lighten3B`

Lightens a `ccColor3B`'s color by a certain amount

### `darkenB`

Darkens a `ccColor3B`'s color by a certain amount

### `to3B`

Converts a `ccColor4B` to a `ccColor3B`

### `to4B`

Converts a `ccColor3B` to a `ccColor4B` (with support for choosing the alpha)

### `to4F`

Converts a `ccColor4B` to a `ccColor4F`

### `cc3x`

Converts a number hex value to a `ccColor3B`

### `cc3bFromHexString`

Converts a `ccColor3B` to a hex `std::string`

### `cc4bFromHexString`

Converts a `ccColor4B` to a hex `std::string`

### `cc3bToHexString`

Converts a hex `std::string` to a `ccColor3B`

### `cc4bToHexString`

Converts a hex `std::string` to a `ccColor4B`

## `vectorToCCArray`

Converts a `std::vector` of any type to a `CCArray`

## `ccArrayToVector`

Converts a `CCArray` to a `std::vector`

## `mapToCCDict`

Converts a `std::map` to a `CCDictionary`

## `getMousePos`

Gets the mouse position in cocos2d coordinates. On mobile platforms, this will return (0, 0)

## Web Utilities

### `openLinkInBrowser`

A simple way to open a link in the user's browser.

### `fetch`

Fetches data from a URL and returns it as a `std::string`.

### `fetchFile`

Fetches data as a file from a URL and uses a `ghc::filesystem::path` as a path for the file.

### `fetchJSON`

Fetches data from a URL and parses it as a `JSON::value`