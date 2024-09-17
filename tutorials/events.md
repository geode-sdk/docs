# Events

For most things in GD, such as `CCTextInputNode`, GD and Cocos2d-x use a **delegate-based event system**, where you install a delegate on the target class, and the delegate receives events via overridden virtual functions. This system works fine for most situations, but is often quite clumsy to use and runs into a few important issues: you have to manually deal with removing the delegate if the target class outlives the delegate, and more importantly, you can only have one delegate per target.

As an alternative to this system, Geode introduces **events**. Events are essentially just small messages broadcast across the whole system: instead of having to install a single delegate, an unlimited number of classes can listen for events. The target that emits the events does not need to have any knowledge of its consumers; it just broadcasts events, and any receivers there are can handle them.

Events are primarily interacted with through three classes: [`Event`](/classes/geode/Event), [`EventListener`](/classes/geode/EventListener), and [`EventFilter`](/classes/geode/EventFilter). `Event` is the base class for the events that are being broadcast; `EventListener` listens for events, and it uses an `EventFilter` to decide which events to listen to. Let's explore how they work through **an example**.

Consider a system where one mod introduces a **drag-and-drop API** to Geode, so the user can just drag files over the GD window. Now, this mod itself probably won't know how to handle every single file type ever; instead, it exposes an API so other mods can handle actually dealing with different file types. However, using a delegate-based system here would be quite undesirable - there is definitely more than one file type in existence. While the mod could just have a list of delegates instead of a single one, those delegates have to manually deal with telling the mod to remove themselves from the list when they want to stop listening for events.

Instead, the drag-and-drop API should leveradge the Geode event system by first defining a new type of event based on the `Event` base class:

```cpp
using namespace geode::prelude;

class DragDropEvent : public Event {
public:
    using Files = std::vector<ghc::filesystem::path>;

protected:
    Files m_files;
    CCPoint m_location;

public:
    DragDropEvent(Files const& files, CCPoint const& location);

    Files getFiles() const;
    CCPoint getLocation() const;
};
```

Now, the drag-and-drop mod can post new events by simple creating a `DragDropEvent` and calling `post` on it.

```cpp
void handleFilesDroppedOnWindow(...) {
    ...

    DragDropEvent(...).post();
}
```

That's all - the drag-and-drop mod can now rest assured that any mod expecting drag-and-drop events has received them.

Let's see how that listening part goes, through an example mod that expects [GDShare](https://github.com/hjfod/GDShare-mod) level files and imports them:

```cpp
using namespace geode::prelude;

$execute {
    new EventListener<EventFilter<DragDropEvent>>(+[](DragDropEvent* ev) {
        for (auto& file : ev->getFiles()) {
            if (file.extension() == ".gmd") {
                handleGMDImport(file);

                // This stops event propagation, marking the file as handled. 
                // Stopping propagation is usually not needed, and shouldn't be 
                // done by default, however sometimes you want to stop other 
                // listeners from dealing with the event - such as here, where 
                // importing the same file twice would be undesirable
                return ListenerResult::Stop;
            }
        }
        // Propagate this event down the chain; aka, let other listeners see 
        // the event
        return ListenerResult::Propagate;
    });
}
```

This is all the mod needs to do to set up a **global listener** - one that exists for the entire duration of the mod. Now, whenever a `DragDropEvent` is posted, the mod catches it and can do whatever it wants with it.

This code also uses the default templated `EventFilter` class, which just checks if an event is right type and then calls the callback if that is the case, stopping propagation if the callback requests it to do so. However, we sometimes also want to create custom filters.

For example, let's say another mod wants to use the drag-and-drop API, but instead of always listening for events, it wants to have a specific node in the UI that the user should drop files over. In this case, the listener should only exist while the node exists, and only accept events if it's over the node. We could of course deal with this in the global callback, however we can simplify our code by creating a custom filter that handles accepting the event. We can also include the file types to listen for in the filter itself, simplifying our code even further.

We can create a custom `EventFilter` by inheriting from it:

```cpp
using namespace geode::prelude;

class DragDropOnNodeFilter : public EventFilter<DragDropEvent> {
protected:
    CCNode* m_target;
    std::unordered_set<std::string> m_filetypes;

public:
    // The callback does not need to return ListenerResult nor take the whole 
    // DragDropEvent as a parameter - if the callback is called, then we know 
    // the event was over the node and the file types were correct already
    using Callback = void(std::vector<ghc::filesystem::path> const&);

    ListenerResult handle(MiniFunction<Callback> fn, DragDropEvent* event);
    DragDropOnNodeFilter(CCNode* target, std::unordered_set<std::string> const& types);
};
```

For the implementation of `handle`, we need to check that the event occurred on top of the target node:

```cpp
ListenerResult DragDropOnNodeFilter::handle(MiniFunction<Callback> fn, DragDropEvent* event) {
    // Check if the event happened over the node
    if (m_target->boundingBox().containsPoint(event->getLocation())) {
        // Filter out only file types we can accept
        std::vector<ghc::filesystem::path> valid;
        for (auto& file : event->getFiles()) {
            if (m_filetypes.contains(file.extension().string())) {
                valid.push(file);
            }
        }
        fn(valid);
        // Mark dropped files as handled
        return ListenerResult::Stop;
    }
    // Otherwise let other listeners handle it
    return ListenerResult::Propagate;
}
```

Now, to install an event listener on a specific node, we have two options. If the node is our own class, we can just add it as a class member:

```cpp
class DragDropNode : public CCNode {
protected:
    EventListener<DragDropOnNodeFilter> m_listener = {
        // You can bind member functions as event listeners too!
        this, &DragDropNode::onDrop,
        // The filter requires some args so we have to explicitly construct it
        DragDropOnNodeFilter(this, { ".gmd", ".gmd2", ".lvl" })
    };

    void onDrop(std::vector<ghc::filesystem::path> const& files) {
        // Handle dropped files
    }
};
```

When `DragDropNode` is destroyed, the listener is automatically destroyed and unregistered aswell, so you don't need to do anything else.

However, using a member function is not always possible. For example, if you're hooking a class, [event listeners don't work in fields](/tutorials/fields.md#note-about-addresses); or if you want to listen for events on an existing node whose class you don't control.

In these cases, there exists a Geode-specific helper called [`CCNode::addEventListener`](/classes/cocos2d/CCNode#addEventListener). You can use this to **add event listeners to any node** - including existing ones by GD!

```cpp
auto dragDropNode = CCNode::create();
dragDropNode->addEventListener<DragDropOnNodeFilter>(
    [dragDropNode](auto const& files) {
        // Handle dropped files
    },
    { ".gmd", ".gmd2", ".lvl" }
);
```

`addEventListener` is meant only for events that are have a target node - it assumes that the first parameter of the filter's constructor takes `this` as the argument. Other parameters to the filter's constructor, such as the file types here, can be passed as the rest of the argument list to `addEventListener`.

Any event listener added with `addEventListener` is automatically destroyed aswell when the node is destroyed. You can also provide a string ID for the event listener as the first argument to `addEventListener`, and then manually remove the listener later using [`removeEventListener`](/classes/cocos2d/CCNode#removeEventListener).

## Dispatched events

There also exist special types of events called **dispatch events** - these are intended for use within optional dependencies. See [the tutorial on dependencies](/mods/dependencies.md#events) for more information.
