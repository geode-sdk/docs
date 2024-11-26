# Hook Priority

Hook priority allows for a developer to control when their hook will be called in relation to other hooks on the same function. When done well, hook priority can improve a mod's compatibility. However, poorly applied priorities can also completely break compatibility with other mods.

> :warning: This is advanced functionality! Most developers will not need to touch this system.

The syntax, as previously mentioned in the [Hooking tutorial](/tutorials/modify.md), looks like this:

```cpp
class $modify(cocos2d::CCLabelBMFont) {
    static void onModify(auto& self) {
        if (!self.setHookPriorityPost("cocos2d::CCLabelBMFont::init", Priority::First)) {
            geode::log::warn("Failed to set hook priority.");
        }
        if (!self.setHookPriorityPre("cocos2d::CCLabelBMFont::setString", Priority::Late)) {
            geode::log::warn("Failed to set hook priority.");
        }
        if (!self.setHookPriorityAfterPost("cocos2d::CCLabelBMFont::limitLabelWidth", "geode.node-ids")) {
            geode::log::warn("Failed to set hook priority.");
        }
    }
};
```

A function with no explicit priority is given the priority of `Priority::Normal`.

There are 7 pre-assigned priority values recommended for use: `First, VeryEarly, Early, Normal, Late, VeryLate, Last`. Arithmetic on these values are also possible, although not recommended: `Priority::Early + 2`, meaning 2 internal units later than `Early`.

There are also 2 prefixes related with priorities: `Pre, Post`. Pre priorities sort based on code called **before** the original call, and Post priorities sort based on code called **after** the original call.

Here's an example of the pre/post differentiation:

```cpp
void functionEarlyPre() {
    log::info("statement 1");
    function();
}
void functionLatePre() {
    log::info("statement 3");
    function();
}
void functionNormal() {
    log::info("statement 2");
    function();
    log::info("statement 5");
}
void functionEarlyPost() {
    function();
    log::info("statement 4");
}
void functionLatePost() {
    function();
    log::info("statement 6");
}
```

## Guidelines

**Always use** the before/after functions instead of doing arithmetic on priorities! Unless circular priorities happen, it is guaranteed for them to work consistently even if the other mods change their priorities.

- If you want to run your code after some other mod's code before calling original, use `setHookPriorityAfterPre`.

- If you want to run your code before some other mod's code and before calling original, use `setHookPriorityBeforePre`.

- If you want to run your code before some other mod's code after calling original, use `setHookPriorityBeforePost`.

- If you want to run your code after some other mod's ccodeand after calling original, use `setHookPriorityAfterPost`.

Using raw numbers as priorities is **strongly discouraged** for mod compatibility.
There are tools for mod compatibilities! Please take advantage of them.

- If you want your function to run _before_ all other code, including the original function, use `Priority::First` with `setHookPriorityPre` and call the original function _after_ your code.

- If you want your function to run _after_ all other code, use `Priority::Last` with `setHookPriorityPost` and call the original function at the start of your hook.

- If you want your function to run _after_ the original function, but _before_ other hooks, use `Priority::First` with `setHookPriorityPost` and call the original function at the start.

- If you're _reimplementing_ the original function and do not call the original, use `Priority::Last` with `setHookPriorityPre`.

Obviously, these are not strict rules. Feel free to choose the hook priority that best fits your situation.

## Notes

This section details some additional tricks to the hook priority system.

- Be sure you have the function name correct in `setHookPriority`! The namespace must be included.

- While the current priority is preserved across function calls, it is not preserved in other situations. For example:

  ```cpp
  void hook_fn() {
      Loader::get()->runInMainThread([]() {
          fn(); // <-- this would call hook_fn
      });
  }
  ```

  Instead of calling the original, this will call the hook again. This also applies for calling the original through a Task (see [geode#994](https://github.com/geode-sdk/geode/issues/994)).

  Calling the original through another function will work as long as it is in the same thread:

  ```cpp
  void x() {
      fn(); // <-- this calls the original, instead of hook_fn
  }

  void hook_fn() {
      x();
  }
  ```

- Using a stub function with `Priority::First` with `Pre` or `Priority::Last` with `Post` is an effective way to disable a function. In this scenario, it becomes impossible for other mods to call the original (as their hooks wouldn't be called either), so this is not recommended.

- Pre/Post values embedded into `Priority::` do exist, but they are not that recommended as arithmetic on them can be confusing. (Positive becomes negative for Post)

- Manual hooks can have their priorities set through the [Hook::setPriority](/classes/geode/Hook/#setPriority) method.

- It is not possible to set the priorities of multiple functions with overloaded parameters in the same modify class (as this would require specifying arguments). In this situation, either split the hooks into separate modify classes or manually hook.

## Internal Implementation

The initial call to the function starts with the hook with lowest priority (internally `INT_MIN`).
Each call to the original function increases the current priority, with the original function being at the highest priority (internally `INT_MAX`).
As each function in the chain returns, priority is decreased until the function with lowest priority returns.
At this point, the function call is finished and execution returns to the original caller.

The 7 pre-assigned priorities have the values of `-3000, -2000, -1000, 0, 1000, 2000, 3000` respectively.

An example is shown by this diagram and its accompanying code:

![Diagram detailing the flow of execution for a hooked function](/assets/hook_priority.png)

```cpp
// priority of -1'000'000
void fn_lowest() {
    log::info("statement 1");
    fn();
    log::info("statement 6");
}

// priority of -100
void fn_neg100() {
    // code here happens prior to statement 2 and after statement 1
    fn();
    log::info("statement 5");
}

// priority of 0
void fn_default() {
    log::info("statement 2");
    fn();
    log::info("statement 4");
}

// priority of 1'000'000
void fn_highest() {
    log::info("statement 3");
    fn();
    // code here will happen prior to statement 4 and after the original function call
}
```

The statements will be printed in numerical order, from "statement 1" to "statement 6".

A developer may also choose to not call the original, which means hooks that are a higher priority than the developer's hooks (including the original function) will not be executed.

### Internal notes

- A hook priority of `INT_MAX` will not function as expected, as the original function also has a priority of `INT_MAX`.
