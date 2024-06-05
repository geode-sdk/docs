# Tasks

The Tasks API is used for running **asynchronous** tasks. It handles all the dangers of multi-threading that modding Geometry Dash comes with. At the same time, it uses the `EventListener` API to return the status of execution, the result, a progress type, and whether the task was cancelled or not.

## Creating a Task

Let's say you have to make an expensive calculation inside your mod. For educational purposes, we are just going to iterate 10000000 numbers, and return their sum. Doing this on the main thread would freeze up the game, which isn't what we want. Why not paralellize, and run an asynchronous task to do the job?

Here's our incredibly complex code:

```c++
int sum = 0;
for (int i = 1; i <= 10000000; i++) {
    sum += i;
}
```

Let's first take a look at the Task class:

```c++
template <std::move_constructible T, std::move_constructible P = std::monostate>
class Task final;
```

We can see that it is templated, with 2 arguments. The first one is the **actual return type we will get from the Task**. Depending on whether the Task can fail, you might decide to return a **Result** type. In our case, we will just return an **int**. The second argument is the **progress type of the task**. This can be anything you want, from the simplest int (0-100, like a percentage) to a more complex class. We can use a simple **int** that counts from 0 to 100, like a percentage value, as progress. It is also recommended to **alias your task type**, to prevent code duplication. Only the first argument is required.

```c++
Task<int, int>
// Or, if we're aliasing
using PointlessTask = Task<int, int>;
```

> :warning: If you use a **class** as either the result / progress type, you have to assure that it is **move contstructible**. You can learn more about move semantics [here](https://www.learncpp.com/cpp-tutorial/introduction-to-smart-pointers-move-semantics/). The key principles you have to learn are the different constructor types, `std::move`, and good practices while using these features.

To wrap our calculation inside of a Task, we can use the `Task<int, int>::run()` static method (or `PointlessTask::run()`, if aliased) to create our task. This function takes **2 arguments**.
 - A `MiniFunction<Task<T,P>::Result(MiniFunction<void>(P), MiniFunction<bool>())>` (lambda function). This function, as you may have noticed, takes two arguments. The first one is the **progress** callback, which you can call inside your Task to post updates about the progress of your Task. The second callback can be called to check if our Task has been cancelled.
 - A name, used to name the thread that will run your Task (OPTIONAL).

> From now on, this tutorial will only use the alias to refer to the Task that we are using.

Here's what the final result might look like:

```c++
PointlessTask calculation() {
    return PointlessTask::run([] (auto progress, auto hasBeenCancelled) ->  {
        int sum = 0;
        for (int i = 1; i <= 10000000; i++) {
            // Check for cancellation and early return if the Task has been cancelled.
            if (hasBeenCancelled()) {
                return PointlessTask::Cancel();
            }

            sum += i;

            // Post the progress
            progress(static_cast<int>(std::ceil(i / 10000000 * 100)))
        }

        return sum;
    }, "My Task");
}
```

> :warning: If you want to run asynchronous code inside the Task, you should use `Task::runWithCallback` instead.

Note the usage of the two functions discussed above. We first check if our task has been cancelled. If it hasn't, we continue our normal operation (adding to the sum), then we post our progress to the progress callback. You'll see how we use those 2 values below.

## Listening to our Task

We have created a Task, now we just need to run it, and listen to the result. Notice that we have wrapped the Task inside of a function. This is the recommended way to start a Task, but you can do it any way you want.

To listen for Task events, we need to create an `EventListener`:

```c++
#include <Geode/loader/Event.hpp>

class MyCoolClass {
    // other stuff...
    EventListener<PointlessTask> m_taskListener;
    // other stuff...
};
```

If you are **hooking** a layer, you can add a listener to the Modify class' **fields**:

```c++
class $modify(MenuLayer) {
    struct Fields {
        EventListener<PointlessTask> m_taskListener;
    };
};
```

First of all, we need to **bind** a listener function to our listener. We have 2 choices: a **lambda**, or a **member function**.

```c++
// Lambda
m_taskListener.bind([] (PointlessTask::Event* event) {

});

// Member
m_taskListener.bind(this, &MyCoolClass::onTask);

void MyCoolClass::onTask(PointlessTask::Event* event) {

}
```

Notice that we used the `PointlessTask::Event` type as the argument to our functions. This type is defined by `Task.hpp`, and it contains all the necessary information we need about the running Task. Here is how you should handle it.

```c++
void MyCoolClass::onTask(PointlessTask::Event* event) {
    // Check if we have a value. We will always receive a reference to the result.
    // You can also use a const reference.
    if (int* result = event->getValue()) {
        // The Task completed successfully! Do what you need with the value.
    } else if (std::optional<int> progress = event->getProgress()) {
        // The progress callback was called.
    } else if (event->isCancelled()) {
        // The Task was cancelled
    }
}
```

You can structure your code in any way inside the callback, this is just the way we commonly use the `Event` type.

After binding the function that will handle the Task result, we now need to actually start the task. You can do this with the following:

```c++
// calculation() is the function we created earlier, that returns the Task.
m_taskListener.setFilter(calculation());
```

> :warning: `setFilter` should always be called after `bind`

## Other ways of creating Tasks

There are other ways to create Tasks. We will go summarily through them, you can find out more info about them by **reading the header file**.

### Task::runWithCallback()

`Task::runWithCallback()` is used to create a task the returns its value through another callback (passed as a param). This type of Task can be used if you split the work into other threads inside the Task.

### Task::all()

`Task::all()` is used to create a Task that does nothing but to listen to the result of other tasks. It takes an array of running tasks as parameter. Once all of the tasks finish, the "all" task will return a list with the results of all of the tasks.

### Task::map()

`Task::map()` is used to map (to transform) the result of a Task to another structure. Think of it as the `map()` function you can find in many programming languages.

### Task::immediate()

`Task::immediate()` is used to create a task that will immediately finish with the given value.