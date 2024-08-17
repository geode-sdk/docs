# Hook Priority

Hook priority allows for a developer to control when their hook will be called in relation to other hooks on the same function. When done well, hook priority can improve a mod's compatibility. However, poorly applied priorities can also completely break compatibility with other mods.

> :warning: This is advanced functionality! Most developers will not need to touch this system.

The syntax, as previously mentioned in the [Hooking tutorial](/tutorials/modify.md), looks like this:

```cpp
class $modify(cocos2d::CCLabelBMFont) {
    static void onModify(auto& self) {
        if (!self.setHookPriority("cocos2d::CCLabelBMFont::init", 1'000'000'000)) {
						geode::log::warn("Failed to set hook priority.");
				}
    }
};
```

A function with no explicit priority is given the priority of 0.

The initial call to the function starts with the hook with lowest priority (`INT_MIN`).
Each call to the original function increases the current priority, with the original function being at the highest priority of `INT_MAX`.
As each function in the chain returns, priority is decreased until the function with lowest priority returns.
At this point, the function call is finished and execution returns to the original caller.

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

## Guidelines

Using hook priorities of `INT_MIN` and `INT_MAX - 1` is **strongly discouraged** for mod compatibility.
There are four billion possible priorities! Please take advantage of them.

- If you want your function to run _before_ all other code, including the original function, use a very low hook priority and call the original function _after_ your code.

- If you want your function to run _after_ all other code, use a very low hook priority and call the original function at the start of your hook.

- If you want your function to run _after_ the original function, but _before_ other hooks, use a high hook priority and call the original function at the start.

- If you're _reimplementing_ the original function and do not call the original, use a very high hook priority.

Obviously, these are not strict rules. Feel free to choose the hook priority that best fits your situation.

## Notes

This section details some additional tricks to the hook priority system.

- A hook priority of `INT_MAX` will not function as expected, as the original function also has a priority of `INT_MAX`.

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

- Using a stub function with a very low hook priority is an effective way to disable a function. In this scenario, it becomes impossible for other mods to call the original (as their hooks wouldn't be called either), so this is not recommended.

- Manual hooks can have their priorities set through the [Hook::setPriority](/classes/geode/Hook#setPriority) method.

- It is not possible to set the priorities of multiple functions with overloaded parameters in the same modify class (as this would require specifying arguments). In this situation, either split the hooks into separate modify classes or manually hook.
