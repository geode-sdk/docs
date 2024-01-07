# Declarations & Definitions

One of the first errors new C++ developers always face is something along the lines of `function X is not defined`, or `function X has multiple definitions`. This is a very common issue, as C++ is a very old and unfriendly language that doesn't help with a lot of things.

## Declarations

Unlike a lot of modern languages, C++ doesn't lookahead very often to know what there will be in the file later on. Due to this, you sometimes have to manually tell the compiler know what's coming up, as you can't always define symbols immediately.

A **declaration** is a short piece of code that tells the C++ compiler that something exists somewhere in your codebase:

```cpp
// Function declaration
int someFunction(int a, int b);

// Class declaration
class SomeClass;
```

These are also known as **forward declarations**. They are telling the compiler that a symbol will exist, but you will only define it later on.

Forward declarations are most useful (and also necessary) when dealing with symbols that depend on each other, for example functions that call each other:

```cpp
// Forward declaration of isEven to let the compiler know that it will exist
bool isEven(int a);

bool isOdd(int a) {
    return !isEven(a);
}

bool isEven(int a) {
    return !isOdd(a);
}
```

This example will quite obviously loop indefinitely, but in real-world use cases there are many situations where this pattern is useful.

## Definition

A **definition** is a piece of code that properly tells the compiler what the symbol actually means:

```cpp
// Function definition
int someFunction(int a, int b) {
    return a + b;
}

// Class definition
class SomeClass {
    int m_member;
    std::string m_otherMember;

    // Note that this is a forward declaration of SomeClass::memberFunc, but 
    // the member function is not defined, so you can't use it
    void memberFunc();
};
```

In order to use a symbol, like call a function or instantiate a class, **it must be defined**.

You will sometimes hear about **out-of-line definitions**, which usually refers to class member functions being defined outside of the class definition:

```cpp
// Out-of-line definition of SomeClass::memberFunc
void SomeClass::memberFunc() {
    std::cout << ":teehee:" << std::endl;
}
```

## One-definition rule

Another quirk of C++ that causes new programmers quite a bit of headache is [ODR](https://en.wikipedia.org/wiki/One_Definition_Rule). In essence, what ODR means is that each and every class and function may only have at most one definition.

For example, the following code *will not compile*:

```cpp
void sayHi() {
    std::cout << "Hi mom!" << std::endl;
}

// Compiler errors here due to redefinition of sayHi
void sayHi() {
    std::cout << "Hi dad!" << std::endl;
}
```

Another important thing to note about ODR is that **it applies across all source files**. That is, if you have two different source files:

```cpp
// hi.cpp
void sayHi() {
    std::cout << "Hi mom!" << std::endl;
}

// hello.cpp
void sayHi() {
    std::cout << "Hello mom!" << std::endl;
}
```

**This will also be considered an ODR violation.** Any definition of a C++ symbol in a source file is (usually) automatically visible to all other files, so the compiler will notice you have defined a function in multiple different sources and won't be able to decide which one to use.

However, you will most likely rarely encounter ODR due to source files; rather, the most common source of ODR problems is due to headers.

Say you have the following header, `hi.hpp`:

```cpp
// In hi.hpp
#include <iostream>
#include <string>
void sayHi(std::string const& who) {
    std::cout << "Hi " << who << "!" << std::endl;
}
```

Now you include this header through two other, unrelated headers called `meet.hpp` and `greet.hpp`:

```cpp
// In meet.hpp
#include "hi.hpp"
void meetMom() {
    sayHi("mom");
}
```

```cpp
// In greet.hpp
#include "hi.hpp"
void greetDad() {
    sayHi("dad");
}
```

And now you include both of these in your main source file:

```cpp
// In main.cpp
#include "meet.hpp"
#include "greet.hpp"

int main() {
    meetMom();
    greetDad();
}
```

If you tried to compile this example, you would find that it doesn't work. This is because **the `#include` directive in C++ is essentially just a copy + paste tool**.

If we manually expand the includes, what we see happen is first this:

```cpp
// In main.cpp
// From meet.hpp
#include "hi.hpp"
void meetMom() {
    sayHi("mom");
}
// From greet.hpp
#include "hi.hpp"
void greetDad() {
    sayHi("dad");
}

int main() {
    meetMom();
    greetDad();
}
```

And then continuing on expanding `#include` directives recursively (skipping expanding the inclusions of the standard headers `<iostream>` and `<string>`):

```cpp
// In main.cpp
// From meet.hpp
// From hi.hpp included through meet.hpp
#include <iostream>
#include <string>
void sayHi(std::string const& who) {
    std::cout << "Hi " << who << "!" << std::endl;
}
void meetMom() {
    sayHi("mom");
}
// From greet.hpp
// From hi.hpp included through greet.hpp
#include <iostream>
#include <string>
void sayHi(std::string const& who) {
    std::cout << "Hi " << who << "!" << std::endl;
}
void greetDad() {
    sayHi("dad");
}

int main() {
    meetMom();
    greetDad();
}
```

We will find that we have inadvertedly violated ODR through our header inclusions.

> You might be wondering why the standard headers `<iostream>` and `<string>` don't cause issues despite being included multiple times - this is because they employ all of the solutions listed below.

There are **a few solutions to this issue**. The cases in which to use each one will be explained at the end.

### 1. Using header guards

**Header guards** are a peculiar quirk of C and C++ that basically tell the compiler not to copy + paste the same file multiple times:

```cpp
// In hi.hpp
#pragma once
#include <iostream>
#include <string>
void sayHi(std::string const& who) {
    std::cout << "Hi " << who << "!" << std::endl;
}
```

This now expands to:

```cpp
// In main.cpp
// From meet.hpp
// From hi.hpp included through meet.hpp
#include <iostream>
#include <string>
void sayHi(std::string const& who) {
    std::cout << "Hi " << who << "!" << std::endl;
}
void meetMom() {
    sayHi("mom");
}
// From greet.hpp
// hi.hpp is not included through greet.hpp as the compiler notices it has already been included
void greetDad() {
    sayHi("dad");
}

int main() {
    meetMom();
    greetDad();
}
```

You may also sometimes see header guards that look like this:

```cpp
#ifndef __HI_HPP__
#define __HI_HPP__
#include <iostream>
#include <string>
void sayHi(std::string const& who) {
    std::cout << "Hi " << who << "!" << std::endl;
}
#endif
```

This is an old and archaic way of achieving header guards that is still sometimes used because `#pragma once` is *technically* non-standard (every C++ compiler you will ever use supports it, but some very rare ones might not).

This is **not a proper solution** to the issue of multiple definitions though, as if another source file includes `hi.hpp` through some header it will copy + paste the definition aswell and then `sayHi` is once again defined multiple times.

### 2. Defining in source

Whereas there may only be one definition of a symbol, there may be any number of declarations of it. Replacing `hi.hpp` with just a declaration will fix our header inclusions:

```cpp
// In hi.hpp
#include <string>
void sayHi(std::string const& who);
```

Now our headers expand to:

```cpp
// In main.cpp
// From meet.hpp
// From hi.hpp included through meet.hpp
#include <string>
void sayHi(std::string const& who);
void meetMom() {
    sayHi("mom");
}
// From greet.hpp
// From hi.hpp included through greet.hpp
#include <string>
void sayHi(std::string const& who);
void greetDad() {
    sayHi("dad");
}

int main() {
    meetMom();
    greetDad();
}
```

Now we have only **declared** `sayHi` twice, essentially just telling the compiler twice that there will be a `sayHi` function defined later on, which is fine by it. However, this still won't compile, as now we don't have any definition for the `sayHi` function in our code, and **every function you use must be defined**.

The solution is to add the definition of the function to a source file, in this case our main one:

```cpp
// In main.cpp
#include "meet.hpp"
#include "greet.hpp"

void sayHi(std::string const& who) {
    std::cout << "Hi " << who << "!" << std::endl;
}

int main() {
    meetMom();
    greetDad();
}
```

Now this expands to the following code that will compile:

```cpp
// In main.cpp
// From meet.hpp
// From hi.hpp included through meet.hpp
#include <string>
void sayHi(std::string const& who);
void meetMom() {
    sayHi("mom");
}
// From greet.hpp
// From hi.hpp included through greet.hpp
#include <string>
void sayHi(std::string const& who);
void greetDad() {
    sayHi("dad");
}

void sayHi(std::string const& who) {
    std::cout << "Hi " << who << "!" << std::endl;
}

int main() {
    meetMom();
    greetDad();
}
```

### 3. Using `inline` (and `static`)

Another solution is to use the `inline` keyword, which tells the compiler that if this same function definition is encountered multiple times, that's not an accident and it shouldn't error about it:

```cpp
#include <iostream>
#include <string>
inline void sayHi(std::string const& who) {
    std::cout << "Hi " << who << "!" << std::endl;
}
```

Which expands into: 

```cpp
// In main.cpp
// From meet.hpp
// From hi.hpp included through meet.hpp
#include <iostream>
#include <string>
inline void sayHi(std::string const& who) {
    std::cout << "Hi " << who << "!" << std::endl;
}
void meetMom() {
    sayHi("mom");
}
// From greet.hpp
// From hi.hpp included through greet.hpp
#include <iostream>
#include <string>
inline void sayHi(std::string const& who) {
    std::cout << "Hi " << who << "!" << std::endl;
}
void greetDad() {
    sayHi("dad");
}

int main() {
    meetMom();
    greetDad();
}
```

This will compile, as the presence of `inline` on the declaration makes the compiler allow multiple definitions of `sayHi`.

> There is also another keyword called `static` that can achieve visually the same thing as `inline`, but what it does is a little bit different and its use should generally be avoided unless you know what you're doing.

### Best practices

You should **always use header guards** in all *header files* (`.hpp` and `.h`), regardless of what those headers contain.

Whether or not you should use `inline` or define in source depends on the situation, though **it's usually best to define in source**. This may prove a bit of a headache when exporting functions from a DLL, which is left as a problem for another tutorial.

## Templates

Another thing that causes headache is that **templated functions and classes** do not behave as one might expect when it comes to definitions.

For example, if the `sayHi` function was templated:

```cpp
// In hi.hpp
#pragma once
template<class T>
void sayHi(T const& who) {
    std::cout << "Hi " << who << "!" << std::endl;
}
```

If you compiled this now, you would find that despite the function definition not being marked `inline`, this would work just fine even if `hi.hpp` was included from multiple source files. This is because templated symbols are only actually defined when they are **instantiated**; meaning `sayHi<const char*>` only exists once you call it in your code with `sayHi("mom")`.

The exact depths of how template function declarations and definitions work is left as an exercise for the reader, as it is not something you typically encounter when you're just starting out with C++.
