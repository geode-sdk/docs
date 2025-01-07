# Events

For most things in GD, such as `CCTextInputNode`, GD and Cocos2d-x use a **delegate-based event system**, where you install a delegate on the target class, and the delegate receives events via overridden virtual functions. This system works fine for most situations, but is often quite clumsy to use and runs into a few important issues: you have to manually deal with removing the delegate if the target class outlives the delegate, and more importantly, you can only have one delegate per target.

As an alternative to this system, Geode introduces **events**. Events are essentially just small messages broadcast across the whole system: instead of having to install a single delegate, an unlimited number of classes can listen for events. The target that emits the events does not need to have any knowledge of its consumers; it just broadcasts events, and any receivers there are can handle them.

Events are primarily interacted with through three classes: [`Event`](/classes/geode/Event), [`EventListener`](/classes/geode/EventListener), and [`EventFilter`](/classes/geode/EventFilter). `Event` is the base class for the events that are being broadcast; `EventListener` listens for events, and it uses an `EventFilter` to decide which events to listen to. Let's explore how they work through **an example**.

## Creating events

Consider a system where one mod introduces a **drag-and-drop API** to Geode, so the user can just drag files over the GD window. Now, this mod itself probably won't know how to handle every single file type ever; instead, it exposes an API so other mods can handle actually dealing with different file types. However, using a delegate-based system here would be quite undesirable - there is definitely more than one file type in existence. While the mod could just have a list of delegates instead of a single one, those delegates have to manually deal with telling the mod to remove themselves from the list when they want to stop listening for events.

Instead, the drag-and-drop API should leveradge the Geode event system by first defining a new type of event based on the `Event` base class:

```cpp
// DragDropEvent.hpp

#include <Geode/loader/Event.hpp> // Event
#include <Geode/cocos/cocoa/CCGeometry.h> // CCPoint

#include <vector> // std::vector
#include <filesystem> // std::filesystem::path

using namespace geode::prelude;

class DragDropEvent : public Event {
protected:
    std::vector<std::filesystem::path> m_files;
    CCPoint m_location;

public:
    DragDropEvent(std::vector<std::filesystem::path> const& files, CCPoint const& location);

    std::vector<std::filesystem::path> getFiles() const;
    CCPoint getLocation() const;
};
```

> :warning: Note that we have structured the event this way (with protected variables and getters) so that it is **read-only**.

Now, the drag-and-drop mod can post new events by simple creating a `DragDropEvent` and calling `post` on it.

```cpp
// Assume those variables actually have useful values
std::vector<std::filesystem::path> files;
CCPoint location = CCPoint { 0.0f, 0.0f };

DragDropEvent(files, location).post();

```

That's all - the drag-and-drop mod can now rest assured that any mod expecting drag-and-drop events has received them.

## Listening to events

Listening to events is done using an `EventListener`. An event listener needs an `EventFilter`, so that it knows what events to listen to. This default EventFilter will pass a **pointer** to the event as the parameter to the callback we have to define. Here is a very simple example that listens to events for the **entire runtime of the game** - a **global listener**, as you might call it.

```cpp
// main.cpp

#include <Geode/DefaultInclude.hpp> // $execute
#include <Geode/loader/Event.hpp> // EventListener, EventFilter

#include "DragDropEvent.hpp" // Our created event

using namespace geode::prelude;

// Execute runs the code inside **when your mod is loaded**
$execute {
    // This technically doesn't leak memory, since the listener should live for the entirety of the program
    new EventListener<EventFilter<DragDropEvent>>(+[](DragDropEvent* ev) {
        for (std::filesystem::path& file : ev->getFiles()) {
            log::debug("File dropped: {}", file);

            // ... handle the files here
        }

        // We have to propagate the event further, so that other listeners
        // can handle this event
        return ListenerResult::Propagate;
    });
}
```

Notice that our callback returns a `ListenerResult`, more specifically `ListenerResult::Propagate`. This tells the event system that this specific event should **propagate** to the next listeners that are expecting this type of event. If you wish to **stop** this propagation from happening (let's say you don't want **.gmd** files to be propagated to other listeners), then you can return `ListenerResult::Stop`.

```cpp
// main.cpp

#include <Geode/DefaultInclude.hpp> // $execute
#include <Geode/loader/Event.hpp> // EventListener, EventFilter

#include "DragDropEvent.hpp" // Our created event

using namespace geode::prelude;

$execute {
    new EventListener<EventFilter<DragDropEvent>>(+[](DragDropEvent* ev) {
        for (std::filesystem::path& file : ev->getFiles()) {
            log::debug("File dropped: {}", file);
            if (file.extension() == ".gmd") {
                log::info("Detected .gmd file: {}", file);

                // Stop event propagation after this listener.
                return ListenerResult::Stop;
            }
        }

        // If no .gmd file was detected, propagate the event further
        return ListenerResult::Propagate;
    });
}
```

This is all the mod needs to do to set up a **global listener** - one that exists for the entire duration of the mod. Now, whenever a `DragDropEvent` is posted, the mod catches it and can do whatever it wants with it.

This code also uses the default templated `EventFilter` class, which just checks if an event is right type and then calls the callback if that is the case, stopping propagation if the callback requests it to do so. However, we sometimes also want to create **custom filters**.

## Creating custom filters

For example, let's say another mod wants to use the drag-and-drop API, but instead of always listening for events, it wants to have a **specific node in the UI that the user should drop files over**. In this case, the listener should only exist **while the node exists**, and only accept events if it's over the node. We could of course deal with this in the global callback, however we can simplify our code by creating a custom filter that handles accepting the event. We can also include the file types to listen for in the filter itself, simplifying our code even further.

We can create a custom `EventFilter` by inheriting from it:

```cpp
// DragDropOnNodeFilter.hpp

#include <Geode/cocos/base_nodes/CCNode.h> // CCNode
#include <Geode/loader/Event.hpp> // EventFilter

#include "DragDropEvent.hpp" // Our event

#include <filesystem> // std::filesystem::path
#include <functional> // std::function
#include <string> // std::string
#include <unordered_set> // std::unordered_set
#include <vector> // std::vector

using namespace geode::prelude;

class DragDropOnNodeFilter : public EventFilter<DragDropEvent> {
protected:
    CCNode* m_target;
    std::unordered_set<std::string> m_filetypes;

public:
    // We HAVE to specify this alias, the EventListener makes use of it.
    // The default EventFilter<DragDropEvent> would send the event itself as the callback argument
    // For the default EventFilter<DragDropEvent>, the Callback alias looks like:
    // using Callback = ListenerResult(DragDropEvent*)
    //
    // In this case, though, we want to filter the files that we need, so we will only use a vector of files.
    using Callback = ListenerResult(std::vector<std::filesystem::path> const&);

    // This method also needs to exist
    ListenerResult handle(std::function<Callback> fn, DragDropEvent* event);
    DragDropOnNodeFilter(CCNode* target, std::unordered_set<std::string> const& types)
        : m_target(target),
        m_types(types) {}
};
```

> :warning: You have a lot of freedom when defining the EventFilter callback. Remember that the default `EventFilter<Event>` is of type `std::function<ListenerResult(Event*)>`, but your callback can look differently.

For the implementation of `handle`, we need to check that the event occurred on top of the target node:

```cpp
ListenerResult DragDropOnNodeFilter::handle(std::function<Callback> fn, DragDropEvent* event) {
    // If the event didn't happen over the node, just propagate the event further
    if (!m_target->boundingBox().containsPoint(event->getLocation())) {
        return ListenerResult::Propagate;
    }

    std::vector<std::filesystem::path> valid;

    // Filter the files to only include valid ones
    for (std::filesystem::path& file : event->getFiles()) {
        if (m_filetypes.contains(file.extension().string())) {
            valid.push_back(file);
        }
    }

    // If there are no valid files, propagate the event further
    if (valid.size() == 0) {
        return ListenerResult::Propagate;
    }

    // Call the EventListener callback and return the ListenerResult that it gives us
    return fn(valid);
}
```

Now, to install an event listener on a specific node, we have two options. If the node is our own class, we can just add it as a class member:

```cpp
// DragDropNode.hpp

#include <Geode/cocos/base_nodes/CCNode.h> // CCNode
#include <Geode/loader/Event.hpp> // EventListener

#include "DragDropOnNodeFilter.hpp" // Our filter

class DragDropNode : public CCNode {
protected:
    EventListener<DragDropOnNodeFilter> m_listener = {
        this, &DragDropNode::onDrop,
        // We defined a constructor with some arguments, so we have to construct our filter now
        DragDropOnNodeFilter(this, { ".gmd", ".gmd2", ".lvl" })
    };

    ListenerResult onDrop(std::vector<std::filesystem::path> const& files) {
        // Handle dropped files

        return ListenerResult::Propagate;
    }
};
```

There are multiple ways to define a callback for our listener. The method above uses a **member function**. We can also define a **lambda**:
```cpp
EventListener<DragDropOnNodeFilter> m_listener = {
    [](std::vector<std::filesystem::path> const& files) {
        // Handle dropped files...

        return ListenerResult::Propagate;
    },
    DragDropOnNodeFilter(this, { ".gmd", ".gmd2", ".lvl" })
};
```

When our `DragDropNode` is destroyed, the EventListener is automatically destroyed and unregistered aswell, so you don't need to do anything else.

However, using a member function is not always possible. For example, if you're hooking a class, [event listeners don't work in fields](/tutorials/fields.md#note-about-addresses); or if you want to listen for events on an existing node whose class you don't control.

In these cases, there exists a Geode-specific helper called [`CCNode::addEventListener`](/classes/cocos2d/CCNode#addEventListener). You can use this to **add event listeners to any node** - including existing ones by GD!

> :warning: Any `EventFilter` that is used in `addEventListener` **must** have their first **constructor param** as a `CCNode*`. The callback **lambda** should be the first argument passed to `addEventListener`, then you have to pass the next **constructor arguments** for your `EventFilter`

```cpp
auto dragDropNode = CCNode::create();
dragDropNode->addEventListener<DragDropOnNodeFilter>(
    [dragDropNode](std::vector<std::filesystem::path> const& files) {
        // Handle dropped files

        return ListenerResult::Propagate;
    },
    { ".gmd", ".gmd2", ".lvl" }
);
```

Any event listener added with `addEventListener` is automatically destroyed aswell when the node is destroyed. You can also provide a string ID for the event listener as the first argument to `addEventListener`, and then manually remove the listener later using [`removeEventListener`](/classes/cocos2d/CCNode#removeEventListener).

## Dispatched events

There also exist special types of events called **dispatch events** - these are intended for use within optional dependencies. See [the tutorial on dependencies](/mods/dependencies.md#events) for more information.
