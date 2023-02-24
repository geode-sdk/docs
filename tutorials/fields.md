# Fields

Say you want to add a property to an existing class. One way you could do this is by storing the property in a global variable like so:

```cpp
int totalJumps = 0;

class $modify(PlayerObject) {
    void pushButton(PlayerButton button) {
        log::info("the player has jumped {} times !", totalJumps);
        totalJumps++;
        PlayerObject::pushButton(button);
    }
};
```

This works; however, it isn't ideal. The counter never resets back down to 0, and the same value is shared across all instances of PlayerObject.

One could solve this problem by using the `setUserData` function in `CCNode` or by utilizing more complicated systems, but both solutions are less than ideal.

Geode provides a better alternative: **fields**. Fields let you add new member variables to modified classes. Take a look at an example:

```cpp
class $modify(PlayerObject) {
    int m_totalJumps;

    void pushButton(PlayerButton button) {
        log::info("The player has jumped {} times !", m_fields->m_totalJumps);
        m_fields->m_totalJumps++;
        PlayerObject::pushButton(button);
    }
};
```

This code works even if you have multiple `PlayerObject`s, and the counter is initialized as 0 for each one, providing an elegant yet simple solution to the problem.

**Fields must be accessed through the `m_fields` member**. If you access them directly through `this`, you will get **undefined behaviour** (most likely a crash).

Fields are declared just like normal member variables, except that **default values must be applied in the constructor**. 

```cpp
class $modify(MyPlayerObject, PlayerObject) {
    int m_totalJumps;

    MyPlayerObject() : m_totalJumps(13) {}

    void pushButton(PlayerButton button) {
        log::info("the player has jumped {} times !", m_fields->m_totalJumps);
        m_fields->m_totalJumps++;
        PlayerObject::pushButton(button);
    }
};
```
