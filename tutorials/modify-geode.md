# Modifying Geode UI

Since Geode v3.4.0, mods are able to also modify the Geode UI itself in limited ways. This is so mods can do things like add extra buttons to their own mod's popup, make their mod logo have particles, etc. This tutorial shows how to use these APIs to modify Geode's own UIs.

## Listening for UI events

Geode currently exposes three UI events through the `<Geode/ui/GeodeUI.hpp>` header: [`ModPopupUIEvent`](/classes/geode/ModPopupUIEvent), [`ModItemUIEvent`](/classes/geode/ModItemUIEvent), and [`ModLogoUIEvent`](/classes/geode/ModLogoUIEvent). These are for being notified of whenever a mod popup is created, a mod list mod item is created, and a mod logo is created respectively.

Note that **all Geode UI events may and will be posted _multiple times_!** This is because Geode UIs often have an initial created "loading" state, and then fetch data from the Geode servers that is updated onto the created node asynchronously. To allow mods to be notified of when this data is loaded, **the UI events are reposted whenever the state of the UI node changes**. For this reason, **mods can never add nodes without checking if they already exist first**.

You can listen to these events by using Geode's events system as usual, using the [`EventFilter<T>`](/classes/geode/EventFilter) helper class. Note that you should always return `ListenerResult::Propagate` to allow other mods to modify the layer as well, like calling the original in a `$modify` hook!

## Guidelines

There are some guidelines on what you are and are not allowed to do when modifying the Geode UI. These are:

1. The mod may **only access nodes by ID or member**. No matching types or indices, if the node the mod wants to modify doesn't have an ID and is not accessible by a member variable or direct class getter, the mod can not edit it
2. The mod must **always check for null** on every node they access. It should never assume the existence of a node, even if it's trivial
3. The mod must **safely handle all possible failure states**, such as missing sprites. This includes failure states of other logic it runs; if the mod makes a web request, it must not crash on unexpected behaviour!
4. If the mod finds any missing IDs, it must **undo any/all of its changes** and not make any more. The recommended way to do this is to first use `querySelector` to grab all the nodes in the scene it intends to modify beforehand, and then returning early if any of them are null
5. The mod must **give all of its own nodes IDs prefixed by its mod ID (note) and check beforehand if its nodes have been added on all events**

These rules are in place because the Geode UI is a highly volatile place that **may change at any time**, and any mod causing the Geode UI itself to crash would make it impossible to disable without safe mode.

> (note): Nodes nested inside other nodes don't need to be prefixed, as long as the topmost parent is prefixed. For example, `my-mod.id/container > button` is completely fair!

## Example

```cpp
#include <Geode/ui/GeodeUI.hpp>

using namespace geode::prelude;

$execute {
    new EventListener<EventFilter<ModLogoUIEvent>>(+[](ModLogoUIEvent* event) {
        if (event->getModID() == "geode.loader") {
            // Remember: no assumptions, even trivial ones!
            if (auto fart = CCSprite::createWithSpriteFrameName("GJ_demonIcon_001.png")) {
                fart->setScaleX(5);
                fart->setScaleY(3);
                // `event->getSprite()` is guaranteed to not be `nullptr` though
                event->getSprite()->addChildAtPosition(fart, Anchor::Center);
            }
        }
        // You should always propagate Geode UI events
        return ListenerResult::Propagate;
    });
    new EventListener<EventFilter<ModItemUIEvent>>(+[](ModItemUIEvent* event) {
        if (event->getModID() == "geode.loader") {
            // Find the Geode nodes you want to access first
            auto menu = event->getItem()->querySelector("developers-menu");
            auto dev = event->getItem()->querySelector("developers-button");

            // Only then do stuff with them
            if (menu && dev) {
                if (auto fart = CCSprite::createWithSpriteFrameName("GJ_demonIcon_001.png")) {
                    fart->setScaleX(4);
                    fart->setScaleY(2);
                    dev->addChildAtPosition(fart, Anchor::Center, ccp(-15, 0));
                }
            }
        }
        return ListenerResult::Propagate;
    });
}
```
