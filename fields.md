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
However, `field_t` has some quirks when it comes to accessing the value of the field. 
- `field_t` values can only be accessed using the access operator `->*`. In other words, `this->*m_field` is required to read or write to the field. 
- Due to the [operator precedence](https://en.cppreference.com/w/cpp/language/operator_precedence) of `->*`, sometimes you will have to wrap the access operator in parenthesis. You can see an example of this in the previous code block (`(this->*m_totalJumps)++`).

If you need a default value, you can just set it like a normal field.
```cpp
class $modify(PlayerObject) {
    field_t<int> m_totalJumps = 13; // Starts the amount of jumps at 13

    void pushButton(PlayerButton button) {
        Interface::mod()->log() << "the player has jumped " << this->*m_totalJumps << " times !" << geode::endl;
        (this->*m_totalJumps)++;
        $PlayerObject::pushButton(button);
    }
};
```
