# Making web requests

Making web requests in C++ sucks, and GD doesn't really help with it. Cocos2d has the `CCHTTPRequest` class, which GD uses to make its web requests, but it's a complicated mess to try to get working and it's not really ergonomical.

To fix this, Geode has the `WebRequest` API, which is much more versatile. All web requests are **asynchronous**, they run in background and don't block GD's UI thread. Internally, Geode uses `curl_multi` for requests, meaning they share the connection pool, DNS caches, etc., all meaning you should prefer to use Geode's web utils over your own.

This guide will require some knowledge of async, so make sure to read the [Async](/tutorials/async) tutorial first.

## Creating a web request

Creating a web request is as simple as creating a new `WebRequest` object, and setting some params on it.

```cpp
#include <Geode/utils/web.hpp>

web::WebRequest req = web::WebRequest();

// Let's add some query params
req.param("one", "two");
req.param("three", "four");

// Maybe set our UserAgent?
req.userAgent("Geode Web Request!");

// For POST / PUT requests, we should probably send a body. Choose whichever suits your needs

// Byte vector (std::vector<uint8_t>)
req.body({ 0x23, 0x24, 0x25 });
// Strings
req.bodyString("Hello, server!");
// JSON (uses any matjson::Value)
// Note that you should probably have some actual data in your json
auto myjson = matjson::Value();
req.bodyJSON(myjson);

// Let's set some headers, shall we?
req.header("Content-Type", "application/json");

// Set a timeout for the request, in seconds
req.timeout(std::chrono::seconds(30));
```

There are more functions, regarding certificate verification, proxy settings. You can find more info about them by looking through the headers!

Now, let's send out request. The API has specific methods for POST, GET, PUT, PATCH. If you want to send a different method, you can use `.send()`. Calling these functions will return a **WebFuture**, which can be awaited or spawned.

```cpp
// We're still using the same 'req' object from the codeblock above...

std::string url = "https://example.org";

// Let's do a POST request
auto future = req.post(url);
```

## Getting the response from our request

If you remember the [Async](/tutorials/async) tutorial, you probably also remember that futures must be either awaited or spawned as a task. The most straightforward way is using a `TaskHolder`:

```cpp
class MyCoolClass {
    async::TaskHolder<web::WebResponse> m_listener;
};

// Continuing our code from above...
m_listener.spawn(
    "Fetching Awesome Data",
    std::move(future), // you can also do `req.post()` directly here instead
    [](web::WebResponse response) {
        // This is called once the request is finished, you can handle the response here
    }
);
```

> :warning: Once the `TaskHolder` goes out of scope, the request is cancelled, and the callback will never be called. You must store the holder, or use `async::spawn()` directly to globally listen to a request.

Once we receive our response, we can check multiple things on it! You can see the list of everything you can do with intellisense, or by checking the headers.

```cpp
// Check if the response status code is 2xx
res.ok();
// Get the actual status code
res.code();
// Get a specific header
res.header("Content-Type");
// Get all headers
res.headers();
// Get the response body as bytes
res.data();
// Get the response body as a string
res.string();
// Get the response body as matjson::Value (json)
res.json();
```

## A full example

Let's do a full example. This simple code will hook MenuLayer::init, run a request, and log the resulting response, as a string.

```cpp
#include <Geode/modify/MenuLayer.hpp>
#include <Geode/utils/web.hpp>
#include <Geode/loader/Event.hpp>

using namespace geode::prelude;

class $modify(MenuLayer) {
    struct Fields {
        TaskHolder<web::WebResponse> m_listener;
    };

    bool init() {
        if (!MenuLayer::init()) {
            return false;
        }

        auto req = web::WebRequest();
        req.onProgress([](web::WebProgress const& p) {
            log::info("progress: {}", p.downloadProgress().value_or(0.f));
        });

        m_fields->m_listener.spawn(
            req.get("https://pastebin.com/raw/vNi1WHNF"),
            [](web::WebResponse res) {
                log::info("{}", res.string().unwrapOr("Uh oh!"));
            }
        );

        return true;
    }
};
```

Now open your game (with the developer console enabled) and check to see if your request works!
