# Async

Since Geode v5.0.0, the task system has been replaced by a true **asynchronous** system, based on the [Arc](https://github.com/dankmeme01/arc) library. Async has many advantages over tasks or regular multithreading:
* Lightweight -> unlike threads, async tasks are cheap to construct; you can spawn **thousands** with relatively little resources
* Cooperative -> instead of your OS choosing to forcibly pause threads, async tasks **cooperatively** suspend and let another task run
* Less boilerplate

Async is **perfect** for any workload that consists of waiting - such as making a request to a server, waiting for a notification from another thread, waiting a few seconds between tasks, etc.

Geode provides some async utilities in the `<Geode/utils/async.hpp>` header, as well a global async runtime used by mods. You are not forced to use it, but there's rarely a good reason not to. 

## Coroutines / Futures / Pollables

Simply put, a **coroutine** is a function that can pause its execution and resume later. Virtually always, coroutines here will have the return type `arc::Future<T>`, and they are defined by the presence of `co_return`, `co_await` or `co_yield` inside their body. Here are some examples of coroutines:

```cpp
arc::Future<> coro1() {
    co_return;
}

arc::Future<int> coro2() {
    co_await arc::sleep(asp::Duration::fromMillis(100));
    co_return 42;
}

auto coro3 = [] -> arc::Future<> {
    co_return;
};

// Note: the example below is not a coroutine, but it is Undefined Behavior
// A coroutine must have at least one `co_xxx` statement inside, even without a return value.
arc::Future<> nonCoro() {
}

// This function is NOT a coroutine, but it's also entirely valid.
// If you have a function that delegates work to another coroutine,
// `return coro()` will have identical effect to `co_return co_await coro()`
arc::Future<> coro4() {
    return coro1();
}

// Though, this case would be invalid, because it is an error to mix
// `return` and `co_await`/`co_return` in one function.
arc::Future<> coro5() {
    int value = co_await coro2();

    return coro1();
}
```

Coroutines are lazy, and don't run any code until **awaited**. To execute another coroutine (and optionally get its result), the `co_await` expression should be used (**only** possible within a coroutine):

```cpp
arc::Future<int> coro1(int z) {
    log::debug("coro1 called");
    co_return z * 2;
}

arc::Future<> coro() {
    arc::Future<int> fut = coro1(2);
    // no evaluation happens until we await it, so the log hasn't been printed yet

    int value = co_await fut;
    // now the body was executed and the coroutine has returned
}

int func() {
    // this will error, func() is not a coroutine
    co_await coro();
}
```

Most async functions you will write yourself are going to be coroutines that return `Future<>`. Arc also has a lower-level concept called **Pollable**, but it's not described here as you will likely not need to use it directly (but see Arc's readme if you need to)

## Tasks

An `arc::Task<>` is an independent unit of execution, similar to an `std::thread`. Unlike a `Future`, which needs to be awaited or polled by another future to make progress, tasks run **independently**. They are an **entry point** to executing async code, since it's impossible to run a future outside of a task.

Here's an example of two tasks, one sending notifications and the other receiving them:
```cpp
arc::Notify notify;

async::spawn([notify] -> arc::Future<> {
    while (true) {
        notify.notifyOne();
        co_await arc::sleep(asp::Duration::fromMillis(100));
    }
});

async::spawn([notify] -> arc::Future<> {
    while (true) {
        co_await notify.notified();
        log::info("Received notification!");
    }
});
```

Here, both tasks will run independently forever and generate a notifiaction every 100ms. This is very useful for things like singleton worker threads, that wait for main thread to signal them about needing to do a certain task.

Tasks can also be awaited by other tasks and sync code, or aborted via the `TaskHandle`:
```cpp
auto handle = async::spawn([] -> arc::Future<int> {
    // yield() is like a tiny sleep
    co_await arc::yield();

    co_return 42;
});

// if we are inside a coroutine, we can await the task and get the output
int value = co_await handle;

// if we are NOT inside a coroutine, we can block until it's done
// *never* use this when inside a runtime, because that causes resource starvation
int value = handle.blockOn();

// if we don't care about the task, we can cancel it, which will cleanup all resources owned by it
handle.abort();
```

While everything above is generic to Arc, Geode also provides some specific utils that are very handy in GD itself. The `geode::async::spawn` function has an overload that takes another function, which will be called on **main thread** once the task finishes, making it very similar to the old `Task::listen`:

```cpp
// For example, here's a simple way to run some code exactly 1 second from now, using async
async::spawn(
    arc::sleep(asp::Duration::fromSecs(1)),
    [] { log::info("one second has passed!"); }
);

// Send a web request and handle response on main thread
async::spawn(
    web::WebRequest().get("https://example.org"),
    [](web::WebResponse resp) {
        FLAlertLayer::create("Status", fmt::to_string(resp.code()), "OK")->show();
    }
);
```

As well as the `async::TaskHolder<T>` class, which lets you tie a task's lifetime to a specific object. This will automatically cancel the task by calling `abort()` on the handle, similar to the `EventListener` in v4.

```cpp
async::TaskHolder<WebResponse> listener;

listener.spawn(
    web::WebRequest().get("https://example.org"),
    [](web::WebResponse value) {
        log::debug("Status: {}", resp.code());
    }
);

// Calling spawn again will cancel the previous task and replace it:
listener.spawn(...);

// Once the listener is destroyed, the task is automatically cancelled.
// But you can always choose to do it manually:
listener.cancel();
```

Tasks can have names, and we recommend setting them to make async debugging easier:
```cpp
auto handle = async::spawn(...);
handle.setName("My Web Task");

// TaskHolder also has a second way of setting the name, directly when spawning
async::TaskHolder<void> listener;
listener.spawn(
    "Useless Task",
    arc::yield(),
    [] {}
);
```

## Best practices

While async is a great and powerful tool, in C++ it's unfortunately easy to misuse it. This part of the page is dedicated for showing how to use async correctly, and what to avoid doing.

### Blocking

Blocking a thread of the async runtime is one of the worst things you can do, as it prevents all other tasks from running. Take this code for example:

```cpp
async::spawn([] -> arc::Future {
    std::this_thread::sleep_for(std::chrono::seconds{1});
});
```

If you run this, you will likely get angry messages like this in your console:
```
[WARN] [Worker 1] task Task @ 0x7c4bc6fe0790 took 1.002s to yield
```

This is because an async runtime has a fixed number of threads, which drive all tasks together. When you invoke an operation that waits *asynchronously*, for example `arc::sleep`, you tell the runtime "Hey, you can run other tasks for now, wake me up when I need to run", and it simply suspends your task. When you invoke a *blocking* operation, you bypass the runtime, steal its thread and don't give other tasks an opportunity to run.

Almost every blocking operation has a non-blocking counterpart in Arc:
* Sleep -> `arc::sleep`
* Locking a mutex -> Use `arc::Mutex` instead
* Semaphores / condition variables -> Use `arc::Semaphore`, `arc::Notify` (`arc::mpsc` or `arc::oneshot` for message channels)
* Networking -> Use `arc::UdpSocket`, `arc::TcpStream`, which are non-blocking

If you must run an operation that blocks and have no way to go around it, use the **blocking thread pool**. This is a built-in feature of Arc that allows you to run tasks that block or do a lot of heavy CPU work, without slowing down the runtime:

```cpp
auto handle = async::runtime().spawnBlocking<uint64_t>([] {
    // simulate some expensive calculation 
    uint64_t x = 1;
    for (int i = 0; i < 1024; i++) {
        x = x * (x + i);
    }
    return x;
});

// The function above now runs in parallel, and cannot be stopped unlike a task
// To get the output value, you have two ways:

// 1. await if inside an async task
uint64_t value = co_await handle;
// 2. block (only if NOT inside the async runtime!)
uint64_t value = handle.blockOn();
```

### Synchronization

As mentioned in the previous part, using `std::mutex` inside async isn't always a good idea because it can lead to **blocking**. But even if you are sure the mutex is uncontended, using a non-async-aware mutex in a coroutine is **dangerous**:
```cpp
std::mutex mtx;

arc::Future<> coro() {
    std::unique_lock lock(mtx);
    log::info("Before yielding");
    co_await arc::yield();
    log::info("After yielding");
}
```

Async tasks are **not bound to a specific thread**, so it's entirely possible for this to print:
```
[arc-worker-2] [Mod]: Before yielding
[arc-worker-0] [Mod]: After yielding
```

This would mean that the mutex is locked on worker thread 2, but unlocked on worker thread 0, which will **almost certainly lead to a crash**. OS primitives like `std::mutex` expect to be unlocked by the same thread that locked them.

There are two solutions to this: either ensure you *never* hold a lock across a `co_await` point, or use an async-aware `arc::Mutex`. Async mutexes are somewhat slower than OS counterparts, but for most tasks it isn't an issue:

```cpp
arc::Mutex<int> mtx;

arc::Future<> coro() {
    auto lock = co_await mtx.lock();
    *lock = 42;

    // Completely safe to yield or do any other async work here
    co_await arc::yield();
}
```

## Enabling features

To reduce compile times, mods by default only include essential features of Arc (core and time utilities). You can opt into using other features by adding these lines in your CMakeLists.txt

```cmake
# Enable all features
set(ARC_FEATURE_FULL ON CACHE BOOL "" FORCE)

# Or, you can choose to enable granularly
set(ARC_FEATURE_NET ON CACHE BOOL "" FORCE)
set(ARC_FEATURE_TIME ON CACHE BOOL "" FORCE)
set(ARC_FEATURE_SIGNAL ON CACHE BOOL "" FORCE)
set(ARC_FEATURE_DEBUG ON CACHE BOOL "" FORCE)


# All the lines above should go *before* this line in CMakeLists.txt:
add_subdirectory($ENV{GEODE_SDK} ${CMAKE_CURRENT_BINARY_DIR}/geode)
```
