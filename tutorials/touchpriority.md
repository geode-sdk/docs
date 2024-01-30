# Touch Priority

You probably had this issue of "Why does my button not work?" whenever you added a button into a popup before. If not, consider yourself lucky. This tutorial will cover how Geometry Dash handles touch priority, and how you should use it.

## What is touch priority?

Every layer that has `setTouchEnabled(true)` has an assigned touch priority to it. This priority is used to determine which layer will consume the given touch. 

This priority value is completely independent of the Z order of the layer, which would be the preferred way of doing this kind of thing. But we need to deal with what we have.

The smaller the priority value is, the higher its priority. This means a priority of -1 will be checked first than a priority of 0.

## How Geometry Dash uses touch priority

Disregarding all popups, Geometry Dash touch priority is completely equivalent to how Cocos2d handles it. Every layer has touch priority 0 by default. `CCMenu`'s have a touch priority of -128. Other than these, Robtop classes such as `CCTextInputNode` and `SliderTouchLogic` have a touch priority of -500.

### Force priority 

If you've been in the Geometry Dash modding community enough, you've probably heard of this term at least once. Force priority is the system Robtop implemented into Cocos2d in order to handle his popups. One of the most hated additions of Geometry Dash solely because how unintuitive it is.

There are two values related to force priority: `forcePrio` and `targetPrio`

Force priority tracks the current force priority applied to the popup. Popups (mainly `FLAlertLayer` but other popups do it as well) change this value to set the current force priority. It starts from -500 with a single popup, and decreases by 2 for each consequent popup.

Target priority tracks the target priority that should apply to normal utilities such as `CCTextInputNode` or `SliderTouchLogic` (but not `CCMenu`). If the touch priority requested by these classes are bigger value than the target priority, they are set to the target priority instead. Target priority values are the same as the force priority values.

Robtop has 3 main functions for dealing with force priority: `CCTouchDispatcher::registerForcePrio`, `CCTouchDispatcher::unregisterForcePrio`, `CCTouchDispatcher::addPrioTargetedDelegate`. Registering decreases the target priority, unregistering increases the target priority. Adding a prio targeted delegate allows the layer to use the target priority value instead of the requested touch priority.

## How to use it myself

If you're not dealing with any popups, chances are you don't need to deal with any of these either. But if you are, you should follow these guidelines.

If your popup subclasses `geode::Popup<>`, the registering and unregistering is handled automatically. The class also uses the `geode::cocos::handleTouchPriority` utility, which recursively sets the priorities of every children a lower value than itself, handling the cases like adding `CCMenu`'s, which Robtop does not use the prio targeted delegate for.

If you're adding new touch layers after the initiation has been done, you can again call `geode::cocos::handleTouchPriority` for redoing the touch priorities for the new layers.

If you're not using `geode::Popup<>` class and directly subclassing `FLAlertLayer` instead, you should call `FLAlertLayer::int(int opacity)` inside your init. This will handle the registering of the force priority, along with creating the `m_mainLayer`. The `FLAlertLayer::registerWithTouchDispatcher` handles the delegate, and `~FLAlertLayer` handles unregistering of the force priority.

If you don't want to use the geode utility and set the priorities manually, you can call `CCLayer::setTouchPriority`/`CCMenu::setHandlerPriority` on your layer with the `CCTouchDispatcher::getTargetPrio` value.