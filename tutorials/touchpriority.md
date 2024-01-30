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

Target priority tracks the target priority that should apply to layers such as `CCTextInputNode` or `SliderTouchLogic` and `CCMenu`. If the touch priority requested by these classes are bigger value than the target priority, they are set to the target priority instead. Target priority values are the same as the force priority values.

Robtop has 3 main functions for dealing with force priority: `CCTouchDispatcher::registerForcePrio`, `CCTouchDispatcher::unregisterForcePrio`, `CCTouchDispatcher::addPrioTargetedDelegate`. Registering decreases the target priority, unregistering increases the target priority. Adding a prio targeted delegate allows the layer to use the target priority decreased by 1 if force priority is active.

## How to use it myself

If you're not dealing with any popups, chances are you don't need to deal with any of these either. But if you are, you should follow these guidelines.

If your popup subclasses `geode::Popup<>`, the registering and unregistering is handled automatically.

If you're not using `geode::Popup<>` class and directly subclassing `FLAlertLayer` instead, you should call `FLAlertLayer::init(int opacity)` inside your init. This will handle the registering of the force priority, along with creating the `m_mainLayer`. You should also override `registerWithTouchDispatcher` and call `CCTouchDispatcher::addTargetedDelegate` in it, since that will allow you to register the popup not as a prio targeted delegate. If you leave it the default, `FLAlertLayer` will have one less priority than your layers (-503 vs -502), meaning none of the touches will go to your layers since they will be consumed. And lastly, `~FLAlertLayer` handles unregistering of the force priority.

If you want to set the priorities manually (such as for an overlay), you can call `CCLayer::setTouchPriority`/`CCMenu::setHandlerPriority` on your layer with the `CCTouchDispatcher::getTargetPrio` value.