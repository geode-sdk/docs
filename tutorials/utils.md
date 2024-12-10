# Geode Utils

Here is a list of all the utilities that Geode provides.

## Platform related

### geode::addresser

```cpp
// Get the address of a non-virtual function
auto address1 = addresser::getNonVirtual(&MenuLayer::onMoreGames);
// Get the address of a virtual function
auto address2 = addresser::getVirtual(&MenuLayer::init);
```

### geode::cast

```cpp
float f = 3.14f;

// Performs a union cast, UB!
auto p1 = cast::union_cast<int>(f);
// Performs a type punning cast, UB!
auto p2 = cast::reference_cast<int>(f);

// Performs a dynamic cast void* with a static cast
auto p3 = cast::base_cast<CCNode*>(obj);
// Performs a exact type match cast
auto p4 = cast::exact_cast<CCNode*>(obj);
// Performs a dynamic cast that works with GD classes
auto p5 = cast::typeinfo_cast<CCNode*>(obj);
```

### geode::utils::file

```cpp
// readJson and readBinary also exist
auto data1 = GEODE_UNWRAP(file::readString("file.txt"));
// Reading json into a struct
auto data2 = GEODE_UNWRAP(file::readFromJson<MyStruct>("serialized.json"));

// writeBinary also exists
file::writeString("file.txt", "Hello, World!");
// Writing a struct to json
file::writeToJson("serialized.json", MyStruct());

// createDirectoryAll also exists
file::createDirectory("dir");
auto files = GEODE_UNWRAP(file::readDirectory("dir"));
```

`Zip` and `Unzip` classes exist

```cpp
file::openFolder("dir");

// File picking, check the enum FilePickMode and class FilePickOptions
auto task1 = file::pick(mode, options);
auto task2 = file::pickMany(options);

// Combined with FileWatchEvent and FileWatchFilter
GEODE_UNWRAP(file::watchFile("file.txt"));
```

### geode::utils::clipboard

```cpp
auto written = clipboard::write("Hello, World!");
auto read = clipboard::read();
```

### geode::utils::game
    
```cpp
game::exit();
game::restart();
```

### geode::utils::thread

```cpp
auto name = thread::getName();
thread::setName("MyThread");
```

### geode::utils::permission

```cpp
auto status = permission::getPermissionStatus(Permission::ReadAllFiles);
permission::requestPermission(Permission::ReadAllFiles);
```

### Abortion

```cpp
utils::terminate("Error message");
utils::unreachable();
```

### geode::utils::web

TODO: this ones long just check `Geode/utils/web.hpp`

### ObjcHook

```cpp
// Hooking a method
auto hook1 = GEODE_UNWRAP(ObjcHook::create("EAGLView", "initWithFrame:", &MyFunc));
auto hook2 = GEODE_UNWRAP(ObjcHook::create("EAGLView", "initWithFrame:", &MyFunc, &emptyFunc));
```

## Tasks

Visit the [tasks](tasks.md) page for more information.

## Cocos related

### Ref and WeakRef

A shared_ptr and weak_ptr like class for GD objects.

```cpp
Ref<CCNode> m_node;
```

### EventListenerNode

A class that listens for events. It is automatically removed when the node is removed.

### ObjWrapper

A class that wraps a GD object.

### geode::cocos

```cpp
// children
getChild
getChildByTagRecursive
findFirstChildRecursive
getChildBySpriteFrameName
getChildBySpriteName
// safe check
nodeOrDefault
isSpriteFrameName
isSpriteName
nodeIsVisible
// sizes
calculateNodeCoverage
calculateChildCoverage
limitNodeSize
limitNodeWidth
limitNodeHeight
// scene
switchToScene
// textures
reloadTextures
// file
fileExistsInSearchPaths
// color
ccDrawColor4B
invert4B
invert3B
lighten3B
darken3B
to3B
to4B
to4F
cc3bFromHexString
cc4bFromHexString
// containers
vectorToCCArray
vectorToCCArray
ccArrayToVector
mapToCCDict
mapToCCDict
getMousePos
```

### CCArrayInserter

A `std::back_inserter` like class for CCArrays.

### CCArrayExt

A wrapper for CCArray that provides iteration and indexing.

### CCDictionaryExt

A wrapper for CCDictionary that provides iteration and indexing.

### CCMenuItemExt

A wrapper for CCMenuItem that provides more useful constructors like taking a lambda.

### CallFuncExt

A wrapper for CallFunc that takes a lambda.

### Touch priority

```cpp
// recursively sets the touch priority of the children based on itself	
handleTouchPriority(this)
```

## Miscellaneous

### Version stuff

`VersionInfo`, `ComparableVersionInfo` and `semverCompare`

### Timer stuff

`Timer` and `LogPerformance`

### geode::utils::string

```
wideToUtf8
utf8ToWide
toLower
toUpper
replace
split
join
contains
containsAny
containsAll
count
trimLeft
trimRight
trim
normalize
startsWith
endsWith
caseInsensitiveCompare
```

### SeedValue

GD headers use them, its mostly transparent to you.

### geode::utils::ranges

```
find
indexOf
move
join
push
concat
remove
filter
reduce
map
min
max
reverse
```

### geode::node_ids

```
setIDSafe
setIDs
switchToMenu
switchChildToMenu
switchChildrenToMenu
detachAndCreateNode
getSizeSafe
```

### geode::utils::map

```
contains
select
selectAll
values
keys
remap
```

### JsonExpectedValue

```cpp
auto json = checkJson(value, "[root]");
// check the class JsonExpectedValue
```

### Random stuff

```
toBytes
getOr
hash
intToHex
numToString
numToAbbreviatedString
numFromString
timePointAsString`
```

### ColorProvider

No idea how to use this one.