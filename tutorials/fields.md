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
    struct Fields {
        int m_totalJumps = 0;
    };

    void pushButton(PlayerButton button) {
        log::info("The player has jumped {} times !", m_fields->m_totalJumps);
        m_fields->m_totalJumps++;
        PlayerObject::pushButton(button);
    }
};
```

This code works even if you have multiple `PlayerObject`s, and the counter is initialized as 0 for each one, providing an elegant yet simple solution to the problem.

> :warning: If you are using old fields behavior, then **fields must be accessed through the `m_fields` member**. If you access them directly through `this`, you will get **undefined behaviour** (most likely a crash).

Fields are declared just like normal member variables, even constructors and destructors work\*. 

```cpp
class $modify(PlayerObject) {
    struct Fields {
        Fields() : m_totalJumps(13) {
            log::debug("Constructed!");
        }
        int m_totalJumps;
        ~Fields() {
            log::debug("Destructed!");
        }
    };

    void pushButton(PlayerButton button) {
        log::info("the player has jumped {} times !", m_fields->m_totalJumps);
        m_fields->m_totalJumps++;
        PlayerObject::pushButton(button);
    }
};
```

## Note about addresses

> :warning: If you are still using the old fields (which do not use the `Fields` struct), those ones are constructed and destructed in a different address than they exist normally (required for space optimization), so **if you have a class that depends on the address of `this` inside the constructor/destructor**, use `std::unique_ptr` to contain the said object. One such example would be Geode's events, since they are registered to a global map in their constructor.

