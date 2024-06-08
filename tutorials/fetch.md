# Making web requests

> :warning: This is the documentation for the new web API. We have left the old documentation at the bottom of the page, for the time being

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
req.json(myjson);

// Let's set some headers, shall we?
req.header("Content-Type", "application/json");

// Set a timeout for the request, in seconds
req.timeout(30);
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

If you remember the [Tasks](/tutorials/tasks) tutorial, you probably also remember that Tasks have to listened to using the [Events](/tutorials/events) system. We can create an `EventListener` for our request and use it to get our response.

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

## The old web APIs

The old web APIs were deprecated as of **v3.0.0-alpha.1**, you can still find them on older versions of Geode. They contain functions resembling a familiar web API: **fetch**. Geode has two types of web requests: **synchronous** and **asynchronous**.

## Synchronous requests

This is the easiest form of web request you can make. Want to fetch some simple data? Use `web::fetch("https://site.url")`. The function returns a result of either a string, or an error message. Similarly, you can use `web::fetchBytes` to get a byte array instead, or `web::fetchFile` to download a file.

Example:

```cpp
#include <Geode/utils/web.hpp>

using namespace geode::prelude;

// download a- uh...
auto res = web::fetch("https://pastebin.com/raw/vNi1WHNF");
if (!res) {
    // handle error
}

// the resulting data :3
auto data = res.value();

```

However, the synchronous web API should **only be used for short, simple data**. For downloading anything larger, it is recommended to use the **asynchronous** API instead.

## Asynchronous requests

Of course, you could just create an `std::thread` and call `fetch` inside that, but that comes with all sorts of thread safety concerns, and you can't interact with the GD UI from another thread. Luckily, Geode's `web::AsyncWebRequest` API is fully **thread-safe** and all its callbacks are run **in the GD thread**, meaning you can safely interact with UI.

Creating an async web request is really simple:

```cpp
#include <Geode/utils/web.hpp>

using namespace geode::prelude;

web::AsyncWebRequest()
    .fetch("https://pastebin.com/raw/vNi1WHNF")
    .text()
    .then([](std::string const& catgirl) {
        // do something with the catgirl :3
    })
    .expect([](std::string const& error) {
        // something went wrong with our web request Q~Q
    });
```

That's it. The web request will then be dispatched, and after it's finished the callback passed to `then` will be run, and on failure the callback passed to `expect` will be run.

There are also callbacks like `progress` for keeping track of the file download, and `cancelled` for checking if the request has been cancelled. The latter is not usually required, as `expect` will be run aswell if the request is cancelled while it is downloading. However, if you're downloading a bunch of files in bulk, a request may be cancelled after it has been finished, in which case you should use `cancelled` to clean up any side-effects the `then` callback caused.

If you want to ensure that a web request is only made once, for example if you want to update a global manager class, you can use the `join` method on `AsyncWebRequest` to make sure only one instance of the same request exists at a time. Join takes a fully arbitary string ID, which can be anything, but you should make sure it's unique.

