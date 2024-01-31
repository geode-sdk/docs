# Pointer Casting

The usual C++, with its type casting and all. But how is this related to GD modding? Well, a lot of things. Delegates, object oriented nature of Cocos, and so on... Let's explore a few examples.

## Known original type

Let's say you have a callback, and in that you want to downcast the sender. You already know the sender type. What should you use?

```cpp
void MyClass::onClick(CCObject* sender) {
	auto node = static_cast<CCNode*>(sender);
	node->setPosition({0, 0});
}
```

You are expected to use static cast when downcasting the objects you already know the type of. 

## Unknown original type

You need to downcast an object, but you don't know its original type. Using your existing C++ knowledge, you might say "Dynamic cast!", but no. Well, not on Windows or MacOS. Since we do not have symbols in those platforms, the compiler recreates typeinfos for the GD classes, and those classes can not match with the ones found in the GD binary. This is why we have this utility called `geode::casts::typeinfo_cast`, which is literally a recreation of `dynamic_cast` that uses typeinfo name to compare the types. The usage is the exact same, but here is an example anyway:

```cpp
void MyClass::onClick(CCObject* sender) {
	if (auto button = typeinfo_cast<CCMenuItemSpriteExtra*>(sender)) {
		button->setSizeMult(1.2f);
	}
}
```

## Callbacks

If you tried to make a button, chances are you used `menu_selector`. But what does that do? It reinterpret casts your member function into a `void (CCObject::*)(CCObject*)`. Since it reinterpret casts it, you should never give it an invalid function, like a `void MyClass::onClick(int)`. It will compile, and it might *techincally* work, but this is very bad. Don't do it.