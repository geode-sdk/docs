# Events

For most things in GD, such as `CCTextInputNode`, GD and Cocos2d-x use a **delegate-based event system**, where you install a delegate on the target class, and the delegate receives events via overridden virtual functions. This system works fine for most situations, but is often quite clumsy to use and runs into a few important issues: you have to manually deal with removing the delegate if the target class outlives the delegate, and more importantly, you can only have one delegate per target.

As an alternative to this system, Geode introduces **events**. Events are essentially just small messages broadcast across the whole system: instead of having to install a single delegate, an unlimited number of classes can listen for events. The target that emits the events does not need to have any knowledge of its consumers; it just broadcasts events, and any receivers there are can handle them.

There are multiple types of events, each with their own unique purpose, such as [`Event`](/classes/geode/Event/), [`GlobalEvent`](/classes/geode/GlobalEvent/), and [`DispatchEvent`](#dispatched-events).

# Examples

## Creating events

Consider a multiplayer game, where every player has a set amount of health. Obviously, since this is Geode, other mods might want to do something when a player has their health changed.
You could write a delegate-like API, where users of the api have to override virtuals and you have to keep track of delegate lifetimes, which can be a hefty amount of code.

Instead, the API could leverage the Geode event system by first defining a new type of `Event`:

```cpp
// PlayerHealthEvent.hpp

#include <Geode/loader/Event.hpp>
#include <string>

using namespace geode::prelude;

struct PlayerHealthEvent : Event<PlayerHealthEvent, bool(int health), void(std::string name)> {
    using Event::Event;
};
```

Notice the template parameters we provided to `Event`. The `PlayerHealthEvent` parameter we passed tells `Event` what type of event it is. The `bool(int health)` parameter acts as the arguments to receive and send. Lastly, the `void(std::string name)` parameter is our **filter**. With them, we can send and listen to events with specific conditions, without the need to define an if statement or logic.

Now, the multiplayer mod can send new health events by simply creating a `PlayerHealthEvent` and calling `send` on it.

```cpp
// the event is created with the filter args we specified in `void(std::string name)`
// 100 is the health from earlier, defined in `bool(int health)`
PlayerHealthEvent("joey").send(100);
```

## Listening to events

Listening to events is done by using the `listen` method in an `Event`.

```cpp
#include <Geode/Geode.hpp>
#include "PlayerHealthEvent.hpp" // Our previously created event

using namespace geode::prelude;

$on_mod(Loaded) {
    // this will only listen to events triggered with name = "joey"
    auto listener = PlayerHealthEvent("joey").listen([](int health) {
        log::info("Joey's health is now {}", health);
        // this is equivalent to return false; or no return at all
        return ListenerResult::Propagate;
    });
    // the listener is scoped, so destroying it will make it stop listening to events,
    // so we leak the it to stop that
    listener.leak();
}
```

Notice that our callback returns a `ListenerResult`, more specifically `ListenerResult::Propagate`. This tells the event system that this specific event should **propagate** to the next listeners that are expecting this type of event. If you wish to **stop** this propagation from happening (let's say you don't want to handle players whose health is an impossible value), then you can return `ListenerResult::Stop`.

```cpp
#include <Geode/Geode.hpp>
#include "PlayerHealthEvent.hpp"

using namespace geode::prelude;

$on_mod(Loaded) {
    auto listener = PlayerHealthEvent("joey").listen([](int health) {
        if (health < 0) {
            return ListenerResult::Stop;
        }
        log::info("Joey's health is now {}", health);

        return ListenerResult::Propagate;
    });
    listener.leak();
}
```

This is all the mod needs to do to set up a **global listener** - one that exists for the entire duration of the mod. Now, whenever a `PlayerHealthEvent` is posted, the mod catches it and can do whatever it wants with it.

## Global Events

If you want an event that can be filtered by the player name, but also want to handle the health event for all players, then you should use `GlobalEvent`. With GlobalEvent, your listener can be unfiltered and all events will be received.

```cpp
// PlayerHealthEventGlobal.hpp

#include <Geode/loader/Event.hpp>

using namespace geode::prelude;

struct PlayerHealthEventGlobal : GlobalEvent<PlayerHealthEventGlobal, bool(int health), void(std::string name)> {
    using GlobalEvent::GlobalEvent;
};
```

Usage is mostly the same, **however**, there is a difference for `listen`:

```cpp
// Both of these are valid:

// Note how the filter args come first in the lambda
PlayerHealthEventGlobal().listen([](std::string const& name, int health) {
    // Handling code goes here..
});

PlayerHealthEventGlobal("joey").listen([](int health) {
    // Handling code goes here..
});
```

## Node bound listeners

You can store a `geode::ListenerHandle` in a class, meaning it will only receive events for as long as the class exists:

```cpp
class MyHealthNode : public cocos2d::CCNode {
    ListenerHandle m_handle;

    bool init() {
        // ...

        m_handle = PlayerHealthEvent("john").listen([this](int health) {
            // ...
        });
        // no need for leak(), the listener handle is alive as long as the node is
    }
};
```

In case of an existing node, or even just for one off listeners, there exists a Geode-specific helper called [`CCNode::addEventListener`](/classes/cocos2d/CCNode#addEventListener). You can use this to **add event listeners to any node** - including existing ones by GD!

## Dispatched events

There also exist special types of events called **dispatch events** - these are intended for use within optional dependencies. See [the tutorial on dependencies](/mods/dependencies#events) for more information.

# Specification