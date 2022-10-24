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

> :warning: For GD mods, you should **not use `dynamic_cast` on Cocos2d nodes**. This is because, due to... problems, `dynamic_cast<ButtonSprite*>(spr)` will **always return null** regardless of what you're expecting. Instead, Geode provides an alternative: `typeinfo_cast`.
