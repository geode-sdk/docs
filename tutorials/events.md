# Events

For most things in GD, such as `CCTextInputNode`, GD and Cocos2d-x use a **delegate-based event system**, where you install a delegate on the target class, and the delegate receives events via overridden virtual functions. This system works fine for most situations, but is often quite clumsy to use and runs into a few important issues: you have to manually deal with removing the delegate if the target class outlives the delegate, and more importantly, you can only have one delegate per target.

As an alternative to this system, Geode introduces **events**. Events are essentially just small messages broadcast across the whole system: instead of having to install a single delegate, an unlimited number of classes can listen for events. The target that emits the events does not need to have any knowledge of its consumers; it just broadcasts events, and any receivers there are can handle them.

There are multiple types of events, each with their own unique purpose, such as [`Event`](https://github.com/geode-sdk/geode/blob/v5.10.1/loader/include/Geode/utils/Task.hpp#L213), [`GlobalEvent`](https://github.com/geode-sdk/geode/blob/v5.10.1/loader/include/Geode/loader/Event.hpp#L1063), and [`DispatchEvent`/`Dispatch`](https://github.com/geode-sdk/geode/blob/v5.10.1/loader/include/Geode/loader/Dispatch.hpp#L15).

## Creating events

Consider a MMORPG system, where every player has a set amount of health. Obviously, since this is Geode, other mods might want to do something when a player has their health changed. If we were to use delegates, this brings up an issue. A delegate-based system for this kind of system would be quite undesirable - there are definitely more names than just "Player". While the mod could just have a list of delegates instead of a single one, those delegate have to manually deal with telling the mod to remove themselves from the list when they want to stop listening for events.

Instead, the MMORPG API should leverage the Geode event system by first defining a new type of `Event`:

```cpp
// PlayerHealthEvent.hpp

#include <Geode/loader/Event.hpp> // Event

#include <string>

using namespace geode::prelude;

class PlayerHealthEvent : public Event<PlayerHealthEvent, bool(int health), std::string> {
public:
    using Event::Event;
};
```

Notice the template parameters we provided to `Event`. The `PlayerHealthEvent` parameter we passed tells `Event` what type of event it is. The `bool(int health)` parameter acts as the arguments to receive and send. Lastly, the `std::string` parameter is our **filter**. With them, we can send and listen to events with specific conditions, without the need to define an if statement or logic.

Now, the MMORPG mod can send new health events by simply creating a `PlayerHealthEvent` and calling `send` on it.

```cpp
PlayerHealthEvent("joey").send(100); // send our event for our listeners to use
```

That's all - the MMORPG mod can now rest assured that any mod expecting health events has received them.

## Listening to events

Listening to events is done by using the `listen` method in an `Event`.

```cpp
// main.cpp

#include <Geode/DefaultInclude.hpp> // $on_mod(Loaded)
#include <Geode/loader/Event.hpp>

#include "PlayerHealthEvent.hpp" // Our previously created event

using namespace geode::prelude;

// on_mod(Loaded) runs the code inside **when your mod is loaded**
$on_mod(Loaded) {
    auto listener = PlayerHealthEvent("joey").listen([](int health) {
        log::info("Joey's health is now {}", health);
        return ListenerResult::Propagate;

        // We have to propagate the event further, so that other listeners
        // can handle this event
        return ListenerResult::Propagate;
    });
    listener.leak(); // We leak the listener to prevent it from being destroyed at the end of the Loaded callback.
}
```

Notice that our callback returns a `ListenerResult`, more specifically `ListenerResult::Propagate`. This tells the event system that this specific event should **propagate** to the next listeners that are expecting this type of event. If you wish to **stop** this propagation from happening (let's say you don't want to handle players whose health is an impossible value), then you can return `ListenerResult::Stop`.

```cpp
// main.cpp

#include <Geode/DefaultInclude.hpp> // $on_mod(Loaded)
#include <Geode/loader/Event.hpp>

#include "PlayerHealthEvent.hpp"

using namespace geode::prelude;

$on_mod(Loaded) {
    auto listener = PlayerHealthEvent("joey").listen([](int health) {
        if (health < 0) {
            return ListenerResult::Stop;
        }
        log::info("Joey's health is now {}", health);
        return ListenerResult::Propagate;

        // We have to propagate the event further, so that other listeners
        // can handle this event
        return ListenerResult::Propagate;
    });
    listener.leak(); // We leak the listener to prevent it from being destroyed at the end of the Loaded callback.
}
```

This is all the mod needs to do to set up a **global listener** - one that exists for the entire duration of the mod. Now, whenever a `PlayerHealthEvent` is posted, the mod catches it and can do whatever it wants with it.

## Global Events

Now, you may be asking, "OK, but what if I want to handle the health event for ALL players?". Don't worry, Geode has you covered with `GlobalEvent`. With GlobalEvent, you don't have to explicitly filter out events. You can create one just like this:

```cpp
// PlayerHealthEventGlobal.hpp

#include <Geode/loader/Event.hpp> // Event

using namespace geode::prelude;

// Notice how we are now using a struct instead of a class.
struct PlayerHealthEventGlobal : public GlobalEvent<PlayerHealthEventGlobal, bool(int health), std::string> {
public:
    using GlobalEvent::GlobalEvent;
};
```

Usage is mostly the same, **however**, there is a difference for `listen`:

```cpp
// Both of these are valid:
PlayerHealthEventGlobal().listen([](int health) {
    // Handling code goes here..
});

PlayerHealthEventGlobal("joey").listen([](int health) {
    // Handling code goes here.. 
});
```

Congrats! You have now made an event that will fire regardless of filters!

## Dispatched events

There also exist special types of events called **dispatch events** - these are intended for use within optional dependencies. See [the tutorial on dependencies](/mods/dependencies#events) for more information.
