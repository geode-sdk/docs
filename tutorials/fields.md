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

Fields are declared just like normal member variables, but inside the special `Fields` struct. Even constructors and destructors work\*. 

> :info: Fields are initialized only whenever they're first accessed, **not** when the modified class is originally created.

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

# External Access

Fields can be accessed outside your modified class like usual, with the help of some casting.

> :warning: Do not use `typeinfo_cast` here, as it isn't actually an instance of `MyGameObject`. 

```cpp
class $modify(MyGameObject, GameObject) {
    struct Fields {
        int m_myField = 2;
    };
};

GameObject* someObject = /*...*/;
// `m_fields` is still required!
static_cast<MyGameObject*>(someObject)->m_fields->m_myField = 12;
```