---
layout: default
title: Adding fields
nav_order: 2
description: "Extending classes and adding fields"
---

# Fields

Say you want to add a property to an existing class. One way you could do this is by storing the property in a global variable like so:

```cpp
int totalJumps = 0;

class $modify(PlayerObject) {
    void pushButton(PlayerButton button) {
        Interface::mod()->log() << "the player has jumped " << totalJumps << " times !" << geode::endl;
        totalJumps++;
        $PlayerObject::pushButton(button);
    }
};
```

This would work, however, it isn't ideal. You may notice that the counter never resets back down to 0, and that the same value is shared with all instances of PlayerObject.

Geode provides a solution to this problem: `field_t`. `field_t` helps provide instance variables to modified classes. Take a look at an example:

```cpp
class $modify(PlayerObject) {
    field_t<int> m_totalJumps;

    void pushButton(PlayerButton button) {
        Interface::mod()->log() << "the player has jumped " << this->*m_totalJumps << " times !" << geode::endl;
        (this->*m_totalJumps)++;
        $PlayerObject::pushButton(button);
    }
};
```
There are a few caviats to know when using `field_t`:
- `field_t` can only be accessed using the access operator `->*`. In other words, `this->*m_field` is required
- Due to the [operator precedence](https://en.cppreference.com/w/cpp/language/operator_precedence) of `->*`, sometimes you will have to wrap the access operator in parenthesis (like in the previous code block)
- `field_t` variables are initialized on first use, not on class creation
- `field_t` variables cannot be assigned a default value in the conventional way of doing so

If you need a default value and can't wait for first use, you can simply hook the constructor of the class and set the value of the field.

```cpp
class $modify(PlayerObject) {
    field_t<int> m_totalJumps;

    void constructor() {
        $PlayerObject::constructor();
        this->*m_totalJumps = 13;
    }
};
```

# Internals

Additional fields internally are stored in a global `address -> container_t<>*` map in the mod, with `container_t<T>` being a templated class that simply stores T and has a virtual destructor that destroys T.

The unique address is determined by taking the address of the `field_t` member. It's value is actually never read, nor should it since it's outside the gd class, so the field is only there as a dummy and to have an unique "id".

To destroy the additional fields, Geode hooks the destructor of the target class via a vtable hook method, taking advantage that classes that inherit CCObject are guaranteed to have a destructor in the third function in the table.
