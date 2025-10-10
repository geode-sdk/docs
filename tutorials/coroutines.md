# Coroutines

[Coroutines](https://en.cppreference.com/w/cpp/language/coroutines) are an underutilized feature of C++20 that not many understand. Under the hood, they are very complex, but luckily Geode makes it quite simple for you. Geode lets you leverage the power of coroutines to write clean asynchronous code, tackle Result propagation, and build Python-style generators with ease.

## Task

For most asynchronous tasks, Geode provides the Task class. See [`Tasks`](/tutorials/tasks) for more information. Any function that returns a Task can be converted into a coroutine by simply using `co_await` on a different task.

Here's an example of a simple coroutine:

```cpp
#include <Geode/utils/web.hpp>
#include <Geode/utils/coro.hpp>

Task<int> getResponseCode() {
    auto res = co_await web::WebRequest().get("https://google.com");

    co_return res.code();
}
```

By using `co_await`, we no longer have to set up an event listener or anything too complicated. The best part is, you can use multiple co_await statements within the same place instead of nesting a thousand callbacks!

There are a few specific things you should be aware of when using this syntax:
* The body of the coroutine is ran in the main thread, possibly only in the next frame.
* If the task the coroutine is waiting on is cancelled, the whole coroutine is cancelled
* If the task returned by the coroutine is cancelled, any pending task that is running is cancelled

You can also send progress values using `co_yield`

```cpp
Task<std::string, int> someTask() {
    for (int i = 0; i < 10; i++) {
        co_yield i;
    }
    co_return "done!";
}
```

## Spawning from regular functions

The correct way to launch a coroutine from a regular function is to use `coro::spawn`. You can do this in a number of ways:

```cpp
// Spawn from a Task via operator<<
coro::spawn << someTask();

// Spawn from a coroutine via operator<<
coro::spawn << [] -> Task<void> {
	co_return;
};

// Spawn from a Task via operator()
coro::spawn(someTask());

// Spawn from a coroutine via operator()
coro::spawn([] -> Task<void> {
	co_return;
});
```

You can use whichever syntax you prefer. It is crucial to use `coro::spawn` on coroutines that return Task to prevent it from canceling when it goes out of scope. In order to prevent Task's `[[nodiscard]]` attribute from getting in the way, using `coro::spawn` on a Task-based coroutine will return a one-item tuple containing the Task. If you wish to get the underlying Task out of the spawn, you can do so simply:

```cpp
auto [task] = coro::spawn << someTask();
```

## $async

Creating a new function for just the asynchronous bits might get tedious. Luckily, you don't have to with the `$async` macro:

```cpp
void logResponseCode(std::string const& url) {
	log::info("Starting request...");

	$async(url) {
		auto req = web::WebRequest();
		auto res = co_await req.get(url);

		log::info("Response code: {}", res.code());
	};
}
```

Under the hood, `$async` sets a coroutine lambda and immediately spawns it via `coro::spawn`. Any arguments within the macro body are lambda captures, so you could just as easily put `=` in there to. It is not recommended to capture by reference as it's not guaranteed that the lambda will finish before the references go out of scope.

## Result propagation

The most annoying part of working with Results is propagation. In Rust, it's as easy as using the `?` operator, but we don't have such nice things in C++. Lucky for us, Geode implements coroutines for Result that allow for easy propagation:

```cpp
#include <Geode/utils/coro.hpp>
#include <Geode/Result.hpp>
#include <matjson.hpp>

// Parse {"myarray": [1, 2, 3]} and return sum
Result<int> parseMyJson(matjson::Value& root) {
    auto name = co_await root.get("myarray");
    auto numbers = co_await name.asArray();

    int output = 0;
    for (auto& number : numbers) {
        output += co_await number.asInt();
    }

    co_return Ok(output);
}
```

Here, `co_await` isn't really awaiting anything at all. Instead it's being used as a way to suspend execution and return if the underlying Result contains an error. If the Result is Ok, it extracts the value and continues execution. This allows you to write freely without having to manually check for errors everywhere.

With the help of another macro, `$try`, you can extend this functionality into non-coroutines as well:

```cpp
// Default value of 0
int parseMyJson(matjson::Value& root) {
    auto res = $try<int> {
        auto name = co_await root.get("name");
        auto numbers = co_await name.asArray();

        int output = 0;
        for (auto& number : numbers) {
            output += co_await number.asInt();
        }
        co_return Ok(output);
    };

    return res.unwrapOr(0);
}
```

Here, once again the `$try` macro sets up a coroutine lambda that gets immediately invoked. This time, since the lambda is guaranteed to be evaluated immediately, `$try` automatically captures everything by reference. The `int` template value represents the Ok output. You can use this to chain multiple Result evaluations together and do error checking as a group.

## Generators

When you need to perform operations on a series of values, it's sometimes preferred to lazily evaluate them instead of collecting everything into a vector. The `coro::Generator` object allows you to create coroutines that lazily yield values, just like Python generators.

Here's what a basic range generator looks like:

```cpp
#include <Geode/utils/coro.hpp>

coro::Generator<int> range(int start, int end) {
	for (int i = start; i < end; ++i) {
		co_yield i;
	}
}

for (int i : range(0, 10)) {
	log::info("My number: {}", i);
}
```

This will log numbers 0 through 9. Generators have a `begin()` and `end()` function that returns an STL-compatible iterator, meaning you can use them with any read-only standard library function that takes in an iterator:

```cpp
auto gen = range(0, 200);
auto sum = std::accumulate(gen.begin(), gen.end(), 0, std::plus()); // Sums all numbers from 0 to 199
```

One benefit of generators over returning vectors is that they can be infinite. You don't need to worry about a massive memory footprint from yielding arbitrary amounts of values, your only roadblock is time.

Here's an example of an infinite fibbonacci generator:

```cpp
#include <Geode/utils/coro.hpp>

coro::Generator<int> fibbonacci() {
	int a = 0;
	int b = 1;
	while (true) {
		co_yield a;
		auto next = a + b;
		a = b;
		b = next;
	}
}


for (int num : fibbonacci()) {
	if (num > 1000) break;
	log::info("My number: {}", num);
}
```

The caller of the generator is the one that determines when the sequence ends, not the callee, giving you lots of flexibility.

Generators have two helper functions that quickly allow you to apply transformations: `map` and `filter`. These transformation functions yield more generators, allowing you to chain them as you please. You can use them on any generator to transform their output like so:

```cpp
// Prints 0, -1, -2, ...
for (int num : range(0, 10).map(std::negate())) {
	log::info("My number: {}", num);
}

// Prints only even numbers
for (int num : range(0, 10).filter([](int n) { return n % 2 == 0; })) {
	log::info("My number: {}", num);
}
```

You can use the `coro::makeGenerator` function to construct a generator based on a vector or a CCArray.
