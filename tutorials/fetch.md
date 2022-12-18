# Making Web Requests

Making web requests in C++ sucks, and GD doesn't really help with it. Cocos2d has the `CCHTTPRequest` class, which GD uses to make its web requests, but it's a complicated mess to try to get working and it's not really ergonomical.

Luckily, Geode come to the rescue with the `geode::utils::web` namespace, which contains functions resembling a familiar web API: **fetch**. Geode has two types of web requests: **synchronous** and **asynchronous**.

## Synchronous requests

This is the easiest form of web request you can make. Want to fetch some simple data? Use `web::fetch("https://site.url")`. The function returns a result of either a string, or an error message. Similarly, you can use `web::fetchBytes` to get a byte array instead, or `web::fetchFile` to download a file.

Example:

```cpp
#include <Geode/utils/fetch.hpp>

USE_GEODE_NAMESPACE();

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
#include <Geode/utils/fetch.hpp>

USE_GEODE_NAMESPACE();

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
`
There are also callbacks like `progress` for keeping track of the file download, and `cancelled` for checking if the request has been cancelled. The latter is not usually required, as `expect` will be run aswell if the request is cancelled while it is downloading. However, if you're downloading a bunch of files in bulk, a request may be cancelled after it has been finished, in which case you should use `cancelled` to clean up any side-effects the `then` callback caused.

If you want to ensure that a web request is only made once, for example if you want to update a global manager class, you can use the `join` method on `AsyncWebRequest` to make sure only one instance of the same request exists at a time. Join takes a fully arbitary string ID, which can be anything, but you should make sure it's unique.

