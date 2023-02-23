# General C++ Tips

This tutorial features some tips for GD modding and C++ programming in general.

## Use `auto`

Despite what some GD modders might tell you, using `auto` is perfectly fine and actually usually desirable. `auto` is the C++-equivalent of the `let` keyword in languages like JavaScript or `var` in C#; it makes C++ **infer the type** of a variable. For example:
```cpp
// inferred as int
auto x = 5;
// inferred as std::string
auto str = functionThatReturnsString();
// inferred as float
auto z = 5.2f;
```
However, do note that `auto` can lead to unexpected behaviour due to **implicit type conversions**. For example:
```cpp
// str is an std::string
std::string str = "Awesome string!";
// Whoops! str is a const char*
auto str = "Awesome string!";
```
This code can not be converted to `auto` by just replacing the type, as this actually calls `std::string`'s `const char*` constructor. The type of string literals is `const char*`, so if you want to create an `std::string` from a string literal, it's usually best to avoid using `auto`. Of course, you could do this:
```cpp
auto str = std::string("Awesome string!");
```
But that's more verbose than the original code you started with, and it's not really more readable.

## Avoid C-style casts

C++ has inherited a lot of technical debt from C, and one of these is C-style casts:
```cpp
float x = 5.3f;
int y = (int)x;
```
You should **avoid using them** for nearly all situations. This is because C-style casts are **wholly unpredictable**. For example, in the above code, what do you thinks happens? Does the float get rounded down to 5? Does the float's binary representation gets reinterpreted as an integer?

C++ provides more explicit alternatives that make it clear what you want:
```cpp
float x = 5.3f;
int y = static_cast<int>(x);
```
The main ones are `static_cast`, `reinterpret_cast`, and `dynamic_cast`. There are also some others like `const_cast`, some casting functions in the standard library like `std::static_pointer_cast` and `std::duration_cast`, and some very magical and terrifying aliases in Geode like `reference_cast` and `as`, but in general these will only be used in specific situations. For most cases, you should just use `static_cast`.

> :warning: A lot of older mods use `reinterpret_cast` for a lot of things, but this is **not recommended at all**. You should always use `static_cast` over `reinterpret_cast`, unless you are **very sure** about what you are doing.

`static_cast` is what you would expect type conversion to do: it converts the value to match the requested type. This works for casting between built-in datatypes, downcasting pointers, casting between your own classes etc..

`dynamic_cast` is a safe version of `static_cast` for polymorphic classes. For example:
```cpp
class A {};
class B : A {};

if (auto b = dynamic_cast<B*>(a)) {
    // b is certainly a valid instance of B
}
```
In this code, `b` will either be certainly a valid `B` or `nullptr`. `dynamic_cast` comes with a **runtime cost** though, so if you know that `b` will definitely be a valid `B`, you can use `static_cast` instead.

> :warning: For GD mods, you should **not use `dynamic_cast` on Cocos2d nodes**. This is because, due to problems, `dynamic_cast<ButtonSprite*>(spr)` will **always return null** regardless of what you're expecting. Instead, Geode provides an alternative: `typeinfo_cast`.

## Avoid memory leaks

C++ does not have a garbage collector or any sort of memory management, so you have to be careful with handling your own memory. In general, the following tips will help:

## Always allocate on the stack when possible

```cpp
struct Thing {
    float x;
    float y;
};

class OtherThing {
    Thing m_thing; // value
};

void someFun(Thing const& thing) {} // pass by const reference
```

If you're passing small structs or classes, always allocate it on the stack by just writing its name. This is the easiest way to keep track of memory; the one built-in memory management C++ does have is RAII, which means that once your struct is no longer used, its memory will be freed. Allocating on the stack is also generally much faster than allocating on the heap.

Allocating objects on the stack also makes dealing with exceptions and early returns easier; no need to add `delete` everywhere, C++ will automatically free the object for you.

```cpp
{
    Thing x;
    // no matter what happens, be it exceptions or 
    // something else, x will be freed at the end
}
```

## Use references over pointers

If you're passing data to a function or a shared parent class to a child, **use references** instead of pointers. References are safer for a lot of reasons, and also don't increment memory. However, when passing by reference, **you have to make sure your class outlives the reference** - for example, **the following code will crash**:

```cpp
struct SomeThing {
    int& m_num;
};

SomeThing* thing;
{
    int x = 5;
    thing = new SomeThing { x };
}
thing->m_num; // Whoops! x is out of scope and so what 
              // m_num is referencing has been deallocated
```

## Be aware of lifetimes

In C++, objects created on the stack live until they go out of scope, and objects created on the heap using `new` live until you call `delete` on them. References and pointers to the object can not extend this lifetime, so make sure that if you're referencing an object, it has not been freed.

For example, some libraries may require you to pass pointers to objects, but you probably want to just pass pointers to stack-allocated objects (since they are automatically freed, so you don't have to worry about calling `delete`)
```cpp
int x;
someLegacyFun(&x); // make sure from someLegacyFun's docs that it only uses the 
                   // pointer to x inside its body, and doesn't store it anywhere
```

## Use `CCObject`-based class over normal ones

Cocos2d comes with its own handy garbage collector, and if you create something like a `CCArray`, its memory will be automatically freed when it's no longer used. Be wary though that this means **you have to make sure any `CCArray` you do actually need doesn't get garbage-collected**. Cocos2d can't know if you've assigned the `CCArray` to a variable or class member, so either use the `Ref` class in Geode to ensure the object stays alive, or manually keep track of it using `retain` if you have to. (**`Ref` should be used in nearly all situations though**)

If you're making your own class that inherits from `CCObject` (for example, a node), **always make sure to call `autorelease`** on it. This will ensure the garbage collector will clean its memory. (Unless you are working with `TableViewCell`s. In that case, never call `autorelease` unless you want to experience the worst bug of all time.)

## Use smart pointers over pointers

If you really do have to use pointers and the class you're working with isn't `CCObject`-based, prefer using **smart pointers** like `shared_ptr` and `unique_ptr` over raw pointers. Smart pointers automatically manage the memory being pointed to, and free it when it's no longer needed. 

## For every call to `new` you have, there must exist a matching call to `delete`

If none of the other strategies are applicable and you have to use raw pointers, the rule of thumb for memory management there is **for every call to `new` you have, there must be a matching call to `delete`**. The only exception to this are **global manager classes**, which are never freed in lieu of being static. Also, in Cocos2d code, you will call `new` for all of your own node classes, but never `delete`; this is because Cocos2d calls `delete` for you when the garbage collector frees the object.
