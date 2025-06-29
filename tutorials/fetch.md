# Making web requests

Making web requests in C++ sucks, and GD doesn't really help with it. Cocos2d has the `CCHTTPRequest` class, which GD uses to make its web requests, but it's a complicated mess to try to get working and it's not really ergonomical.

To fix this, Geode has the `WebRequest` API, which is much more versatile. All web requests are **asynchronous**, so they run on a separate thread.

Under the hood, the `WebRequests` API uses [Tasks](/tutorials/tasks), so make sure you have read that guide before going into this one.

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

Now, let's send out request. The API has specific methods for POST, GET, PUT, PATCH. If you want to send a different method, you can use `.send()`. Calling these functions will return a **WebTask**, which is just an alias for `Task<WebResponse, WebProgress>`.

```cpp
// We're still using the same 'req' object from the codeblock above...

std::string url = "https://example.org";

// Let's do a POST request
auto task = req.post(url);
```

## Getting the response from our request

If you remember the [Tasks](/tutorials/tasks) tutorial, you probably also remember that Tasks have to be listened to using the [Events](/tutorials/events) system. We can create an `EventListener` for our request and use it to get our response.

```cpp
class MyCoolClass {
    EventListener<WebTask> m_listener;
};

// Continuing our code from above...

m_listener.bind([] (web::WebTask::Event* e) {
    if (web::WebResponse* value = e->getValue()) {
        // The request finished!
    } else if (web::WebProgress* progress = e->getProgress()) {
        // The request is still in progress...
    } else if (e->isCancelled()) {
        // Our request was cancelled
    }
});
m_listener.setFilter(task);
```

> :warning: Note that if the task itself goes out of scope, then the request will cancel. If the listener goes out of scope, then the callback will never be called, hence if you want to replace your request, you should cancel the current one, and call setFilter() on the listener instead of creating a new one.

Once we receive our response, we can check multiple things on it! You can see the list of everything you can do with intellisense, or by checking the headers.

```cpp
// Check if the response status code is 2xx
res->ok();
// Get the actual status code
res->code();
// Get a specific header
res->header("Content-Type");
// Get all headers
res->headers();
// Get the response body as bytes
res->data();
// Get the response body as a string
res->string();
// Get the response body as matjson::Value (json)
res->json();
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
        EventListener<web::WebTask> m_listener;
    };

    bool init() {
        if (!MenuLayer::init()) {
            return false;
        }

        m_fields->m_listener.bind([] (web::WebTask::Event* e) {
            if (web::WebResponse* res = e->getValue()) {
                log::info("{}", res->string().unwrapOr("Uh oh!"));
            } else if (web::WebProgress* p = e->getProgress()) {
                log::info("progress: {}", p->downloadProgress().value_or(0.f));
            } else if (e->isCancelled()) {
                log::info("The request was cancelled... So sad :(");
            }
        });

        auto req = web::WebRequest();
        // Let's fetch... uhh...
        m_fields->m_listener.setFilter(req.get("https://pastebin.com/raw/vNi1WHNF"));
        return true;
    }
};
```

Now open your game (with the developer console enabled) and check to see if your request works!
