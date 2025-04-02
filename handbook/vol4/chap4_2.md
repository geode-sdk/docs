# Chapter 4.2: Searching through the docs

At some point or another, you will need to find a class to hook or a function to call. This chapter explains how to search through the geode classes tab.

## Seaching for classes
Geode docs have a useful "classes" tab to search for classes. If you do not know what class something is, it is best to search through devtools first, then search it up the docs. However, if devtools does not tell you the classname, then you would have to find the class yourself.

## I found a class! What now?
Congrats! You can now hook the object by copying the signiture of the function! [Geode's vscode](https://marketplace.visualstudio.com/items?itemName=GeodeSDK.geode) extension provides autocomplete for function signatures, so it is recommended to use that.

## But what if I want to get said object from another object?
Most objects have either a `::get()` or a `::sharedState()` (or even both [Note 1]!) static function. Most of the time, this function will return the object, however if the object does not exist, it will return a nullptr.
c++ lets you check if an object is nullptr and run code if it does exist.

```cpp
if (auto level = GJBaseGameLayer::get()) {
	// If you are in a level, run this code with level being the level
}

else {
	// Otherwise, run this code.
}
```

If the object does not have a `::get()` method, you would need to find another way to get it.

> :warning: Enums do not appear in the search! You would need to find them some other way!

> :warning: Some functions may be inlined on some platforms! Hooking them would cause a compilation error. You would need to find some other function to hook.

> [Note 1] `::get()` and `::sharedState()` methods act the **exact same** on most classes with both of them
