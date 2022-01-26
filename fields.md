---
layout: default
title: Adding fields
nav_order: 2
description: "Extending classes and adding fields"
---

# Fields

Say you want to add a property to an existing class, one way you could do this is by storing the property in a global variable like so:

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

This would work, however, you may notice the counter never resets back down to 0, and that the same value is shared with the second player.

Geode provides a solution: `field_t`

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
However, there are a few caviats to know when using them:
- They can only be accessed using `->*`, so yes, `this->*m_field` is required
- Due to the [operator precedence](https://en.cppreference.com/w/cpp/language/operator_precedence) of ->*, sometimes you will have to wrap it in parenthesis (like in the previous code block)
- They are only initialized on first use, not on class creation
- They cannot be assigned a default value*

If you must have a default value that is also set when the class is created, you can simply do that by hooking the constructor

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