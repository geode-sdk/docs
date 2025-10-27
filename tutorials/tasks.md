# Tasks

The Tasks API is Geode's primary abstraction for running **asynchronous multi-threaded code**. Manually dealing with multi-threaded code in GD is a pain, since you need to make sure all UI and Cocos manipulations happen in the main thread. Tasks abstract this issue away by integrating with the the `EventListener` API, using events to signal when the Task is finished, as well as optionally signaling progress events like download percentage.

## Creating a Task

Let's say you have to make an expensive calculation inside your mod. For educational purposes, we are just going to iterate 10,000,000 numbers, and return their sum. Doing this on the main thread would freeze up the game for a few seconds, which would often be quite undesirable - you don't want your mod to lag the hell out of the game. Using Tasks, we can calculate the sum in another thread using **parallelization**:

Here's our incredible, threaded code:

```cpp
uint64_t sum = 0;
for (uint64_t i = 1U; i <= 10'000'000U; i++) {
    sum += i;
}
```

Let's first take a look at the declaration of the [`Task`](/classes/geode/Task) class:

```cpp
template <std::move_constructible T, std::move_constructible P = std::monostate>
class Task final;
```

First thing you'll notice is that the class takes two template parameters. The first one is the **eventual return type we will get from the Task**. Depending on whether the Task can fail, you might want to return a **Result** type; in our case, since our sum is infallible, we will just return an **uint64_t** directly.

The second template parameter is the **progress type of the task**. This can be anything you want, from a simple integer (0-100, like a percentage) to a more complex class with more detailed progress information. For our purposes, we will just use a simple **int** that counts from 0 to 100, like a percentage value.

It is heavily recommended to **alias your task type**, to prevent code duplication and make for easier refactoring in the future.

```cpp
using SumTask = Task<uint64_t, int>;
```

> :warning: If you use a **class** as either the result / progress type, you have to assure that it is **move constructible**. For most types, this is automatically true. You can learn more about move semantics [here](https://www.learncpp.com/cpp-tutorial/introduction-to-smart-pointers-move-semantics/). The key principles you have to learn are the different constructor types, `std::move`, and good practices while using these features.

To wrap our calculation inside of a Task, we wrap it inside a call to the static method [`Task::run`](/classes/geode/Task#run). It's main argument is a function of type `Task::Result(void(P), bool())`. The first argument passed to this function is the **progress** callback, which you can call inside your Task to post updates about the Task's current state. The second argument is used to check if the Task has been cancelled.

The return type of the callback, `Task::Result`, doesn't really matter - it is essentially a wrapper around the Task's result type, as in `int` for our `SumTask`. It's only speciality is that it allows returning `Task::Cancel` from the function, allowing the Task to cancel itself, and to provide a cheap-to-construct return value if the Task has been cancelled from the outside.

> :information_source: `Task::run` can also be provided a second argument for an arbitary human-readable name for the Task, which can be useful for debugging Task-based code

Here's what our wrapped up Task looks like:

```cpp
SumTask startCalculation() {
    return SumTask::run([](auto progress, auto hasBeenCancelled) -> SumTask::Result {
        uint64_t sum = 0u;
        for (uint64_t i = 1U; i <= 10'000'000U; i++) {
            // Check for cancellation and early return if the Task has been cancelled.
            if (hasBeenCancelled()) {
                return SumTask::Cancel();
            }

            // Do our summing
            sum += i;

            // Post the progress
            progress(static_cast<int>(std::ceil(i / 10'000'000U * 100)))
        }
        return sum;
    }, "My epic task that sums up numbers for some reason");
}
```

Note the usage of the two functions discussed above. We first check if our task has been cancelled. If it hasn't, we continue our normal operation (adding to the sum), then we post our progress to the progress callback. You'll see how we use those 2 values below.

## Listening to our Task

> :warning: **Tasks always need to have a listener.** If no one is listening to a Task, the Task is considered dead and will be cancelled automatically to free up resources.

In order to be notified when a Task finishes, we need create an `EventListener`. Unlike most event-based code where we need to use a special `EventFilter` to listen to events, we don't need to do that for `Task` - they themselves act as their own filter!

```cpp
#include <Geode/loader/Event.hpp>
class MyCoolClass {
    // other stuff...
    EventListener<SumTask> m_sumTaskListener;
    // other stuff...
};
```

If you are **hooking a layer**, you can add a listener to the Modify class' **fields**:

```cpp
#include <Geode/modify/MenuLayer.hpp>
class $modify(MenuLayer) {
    struct Fields {
        EventListener<SumTask> sumTaskListener;
    };
};
```

Internally, Tasks have a handle that uniquely identifies which Task it is. When we call the `setFilter` method on the `EventListener`, we are telling the listener to listen to that Task specifically. There is no way to listen to a bunch of different Tasks at once; you need to group them up to a single Task using `Task::all`.

To fire up our listener, we first need to bind a function to it that is called whenever the Task has something to report:

```cpp
// Assign an anonymous lambda
m_taskListener.bind([](SumTask::Event* event) {
    // The Task has progressed, finished, or been cancelled! Let's handle that!
});

// You can also assign a member function
m_taskListener.bind(this, &MyCoolClass::onSumTask);

void MyCoolClass::onTask(SumTask::Event* event) {
    // Handling code
}
```

To implement the actual business logic of our listener, we need to look at the details of the event. Note that the `Task::Event` type can report three different kinds of states: the Task having finished, the Task having progressed, and the Task having been cancelled. **You should pretty much always handle all three of these states**, unless the Task doesn't have any progress type, or you don't care about intermediate progress.

```cpp
void MyCoolClass::onTask(PointlessTask::Event* event) {
    // Check if we have a value; getValue() always returns a pointer
    if (uint64_t* result = event->getValue()) {
        // The Task completed successfully! Do what you need with the value.
    }
    else if (int* progress = event->getProgress()) {
        // The progress callback was called.
    }
    // This check is technically unnecessary, since Tasks can only ever have 
    // three possible states, but it's good practice to always check it anyway 
    // in case Task gains more states in the future, or if you get rid of the 
    // progress check for example
    else if (event->isCancelled()) {
        // The Task was cancelled
    }
}
```

After binding our callback to the Task listener, we can now actually start the task itself. You can do this by simply assigning the Task as the filter to the listener:

```cpp
// `startCalculation` is the function defined earlier that returns the SumTask
m_taskListener.setFilter(startCalculation());
```

> :warning: You should always call `bind` *before* calling `setFilter`, as if the Task finishes immediately, you will miss the event being posted if you bind afterwards.

## Mapping the value

Often times, Tasks only produce very bare-bones and low-level values themselves, such as a [`WebRequest`](/classes/geode/utils/web/WebRequest) producing a raw HTTP response. This is often times not the type of data you actually want to be working with at the UI level, so you need to **map** the value of the task:

```cpp
SumTask startCalculationHalved() {
    return startCalculation().map(
        [](uint64_t* result) -> uint64_t {
            // Return our result but divided by two
            return *result / 2;
        }
        // We could also define a callback for mapping the progress value, and a 
        // callback for handling the mapped Task being cancelled - by default, 
        // progress is just forwarded as-is
    );
}
```

Note that `Task::map` returns a new `Task`.

`Task::map` can of course also change the type of the Task:

```cpp
Task<std::string> startCalculationString() {
    return startCalculation().map(
        [](uint64_t* result) -> uint64_t {
            // Return our result but as a string
            return std::to_string(*result);
        },
        // Erase any progress information by returning monostate
        [](auto) -> std::monostate {
            return std::monostate();
        }
    );
}
```

> :warning: Cancelling a mapped Task by default also cancels the Task being mapped - in other words, by default all cancels cascade both up and down the chain, unless the caller specifically uses [`Task::shallowCancel`](/classes/geode/Task#shallowCancel)

## Returning pre-calculated values

Often when working with Task-based code, you will encounter a situation where you need to return a Task, but you don't actually need to run any calculations as you have precomputed the value. In this case, while you _could_ use `Task::run` and just immediately return, there is special syntactic sugar just for this case called `Task::immediate`:

```cpp
SumTask startCalculation() {
    // We already know what the sum of 1..10,000,000 is sillyhead! We don't 
    // need to calculate that!
    return SumTask::immediate(49'999'995'000'000u);
}
```

## Waiting for other Tasks / threaded code

Tasks are primarely geared towards running synchronous code in another thread. However, you may sometimes write a Task that needs to wait for another Task or other threaded calculation to finish. For example, maybe instead of writing our number summing logic ourselves, we delegate this responsibility to another library; however, that library's API creates its own thread and takes a callback to be called for the result.

In these cases, you can't really write your Task using `Task::run`, which expects the callback to be synchronous. You could use [`std::condition_variable`](https://en.cppreference.com/w/cpp/thread/condition_variable), but that might be a bit hard to get to work with more complex scenarios. Instead, you can use `Task::runWithCallback`:

```cpp
SumTask startCalculation() {
    return SumTask::runWithCallback([](auto finish, auto progress, auto hasBeenCancelled) {
        // Assuming we are using some external library for summing up numbers 
        // that creates its own thread and calls a callback on finish
        external_library::sumRange(10'000'000U, [finish](uint64_t value) {
            // Note that finish can only be called exactly once; the value can 
            // never be changed, any new progress posted nor the Task cancelled 
            // afterwards, so any code you run past this point can no longer 
            // influence the Task itself.
            finish(value);
            // However, if you do need to run some code here like clean up 
            // temporary files, that is completely fine and safe
        });
    }, "My epic task that sums up numbers for some reason");
}
```

`Task::runWithCallback` is almost identical to `Task::run` - the main difference is that rather than returning the final resulting value, it instead provides a callback function for posting it, akin to JavaScript's `Promise` class. This allows for crafting Tasks that are themselves asynchronous.

## Waiting for a bunch of Tasks to finish

Sometimes we have a bunch of similar Tasks running in parallel, and need to do something after all of them are finished. In these cases, we can use `Task::all`:

```cpp
Task<std::vector<uint64_t*>> startLotsOfCalculations() {
    // SumTask::all takes a vector of Tasks and returns a Task that resolves 
    // into a vector of their results
    return SumTask::all({
        startCalculation(),
        startCalculation(),
        startCalculation(),
        startCalculation(),
        startCalculation(),
    });
}
```

## Lifetime of a Task

The Task lives as long as the handle (the Task object) is not destroyed. If the handle is destroyed, then the task will be marked as **cancelled**. At the same time, you should also not destroy **the listener** while the request is running. Since all callbacks run on the next frame (on the main thread), then if the listener is destroyed before that, **the callback will never be called**.

## Listening to a Task globally

Tasks always need to have a listener. In practice, this means that you should always try to stuff them in an `EventListener` in your layer, or for global Tasks in a manager class. However, sometimes you need to make just a single global request that isn't redone at any point and as such creating a whole manager for it would be kind of overkill. For these situations, you should use `Task::listen`:

```cpp
$execute {
    startCalculation().listen(
        [](uint64_t* value) {
            log::debug("Got a value: {}", *value);
        },
        [](int* progress) {
            log::debug("Progress: {}", *progress);
        },
        []() {
            log::debug("Task was cancelled");
        }
    );
}
```

Note that if your code needs to interact with the Cocos2d UI **at all**, then you should probably be using an `EventListener` in some node or a manager class instead.

## Chaining tasks

You can chain tasks by using `Task::chain`. This allows you to take the result of a task once it finishes and run another task right after, based on that return value. \
Progress updates are only sent from the last Task in the chain, for simplicity

```cpp
Task<std::string> newTask =
    startCalculation()
    .chain([](uint64_t value) -> Task<std::string> {
        // do something expensive with the value idk
        // this is a bad example
        return Task<std::string>::immediate(fmt::format("{}", value));
    });
```

## Coroutines

Tasks can be used in [C++20 coroutines](https://en.cppreference.com/w/cpp/language/coroutines), easily allowing for multiple asynchronous calls to happen within the same code. Note that this may have a little performance overhead compared to regular Task code. See [Coroutines](/tutorials/coroutines) for more information.
