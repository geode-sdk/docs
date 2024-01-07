# Chapter 1.4: Cocos2d

## GD's Game Engine

In order to modify any game, it's good to find out at least one thing: what engine the game was made with. Luckily, GD tells us this on its loading page: **Cocos2d-x**. GD modders have also figured out the specific version of Cocos2d that GD uses: **v2.2.3**. Although, there's a catch.

You see, if we look around the **libcocos2d** file GD comes with and compare it with the [official v2.2.3 branch](https://github.com/cocos2d/cocos2d-x/tree/cocos2d-x-2.2.3/cocos2dx), we will find that there are a bunch of functions and classes present in GD that aren't in the original, like `CCLabelBMFont::limitLabelSize`. As it turns out, **the Cocos2d that GD uses has been modified by RobTop**. These modifications range from small things like a few helper functions added to classes to some entire backends having been rewritten. Due to this, GD mods can't just use the publicly available Cocos2d headers; they need to use modified headers that account for RobTop's changes (which are, naturally, included in Geode).

> :warning: Some old modding tutorials on YouTube might tell you to use libraries such as CappuccinoSDK or Cocos-Headers - these are the traditional 2.1 libraries used, however they are nowadays obsolete.

## But What is Cocos2d?

Cocos2d, or specifically Cocos2d-x 2.2.3, is a **game engine featuring a sprite and node-based UI**. It is what powers GD's user interface, and many of its core data structures. For most GD mods, the two most important things Cocos2d provides us with are the **node system** and **garbage collector**.

The seminal building block of all Cocos2d UI and by extension GD UI is the `CCNode` class. A node is an UI object that can have multiple children, a single parent, and transformations such as position, rotation, scale, etc., with all transformations being also applied to a node's children.

These nodes form a hierarchy known as a **node tree**, on top of which is a `CCScene`. The scene is the only visible node without a parent [[Note 1]](#notes); at a time, there may only be one scene present. The class that manages scenes is `CCDirector`. At all times in GD, there is one director running, which you can get using the `CCDirector::sharedDirector` method, or the Geode-specific `CCDirector::get` shorthand. The director contains information such as the current window size, what scene is running, and getters for many of the other static **singleton managers** in GD. To change to a different scene, you would use the `CCDirector::replaceScene` method. To get the current scene, use `CCDirector::getRunningScene`.

Nearly all nodes you see in GD aren't instances of the base `CCNode` class, but of its derivatives. Some of the most commonly used derivatives are `CCSprite`, `CCMenu`, `CCLabelBMFont`, and `CCLayer`, and some of the most used GD-specific ones are `CCMenuItemSpriteExtra`, `FLAlertLayer`, `GameObject`, and `ButtonSprite`.

As Cocos2d is a node-based framework, nearly all nodes are **aggregates** of other nodes. In other words, **most nodes simply consist of other nodes**. For example, a popup might consist of a node for the background, another node for the text, and maybe some extra nodes for buttons and decoration and such. You most likely **won't have to do any rendering yourself**; if you want to show some text in your node, just create a label and add it as a child to it.

For example, here is **the structure of a comment** in GD:

![Image showing the structure of a CommentCell in GD](/assets/handbook/vol1/CommentCell_dissected.png)

As you can see, the `CommentCell` class consists wholly of other nodes. It does not do any of its own rendering. The position of the nodes (relative to the parent `CommentCell`) is marked in parenthesis; one important thing to note about Cocos2d is that unlike some other game frameworks, **higher Y-coordinate means higher on screen**.

> :warning: Please note that the above image has been simplified and some of the positions do not reflect the actual arrangement of `CommentCell`.

## Creating Nodes

Every node usually has a `create` function for creating an instance of the class, and then a bunch of methods like `setPosition` and `addChild` for setting the node's properties and adding children to it. For example, to create a simple "Hi mom!" text, you would use code like this:

```cpp
// First parameter is the text, second is the font
auto label = CCLabelBMFont::create("Hi mom!", "bigFont.fnt");
// Set the position of the label to be at the coordinates 100, 50 (in units, not pixels)
label->setPosition(100, 50);
// Assuming 'this' is some node aswell
this->addChild(label);
```

By default, all node `create` functions may return `nullptr` in case something goes wrong. However, nodes like `CCLayer` and `CCMenu` are likely to never fail, and in the rare case they do, something has probably already gone catastrophically wrong and the game is about to crash anyway, so handling the null case with these nodes is not usually necessary. However, with some classes like `CCSprite` **handling `create` returning null may be vital**, as `CCSprite::create` will fail if the sprite doesn't exist, which is entirely possible.

## Sprites

Cocos2d is a **sprite-based** framework, meaning that instead of rendering things at runtime using vector graphics, nearly everything shown on screen are textures loaded from disk. If you've ever messed around with GD's files, you've probably noticed this; most of GD's textures are contained in **sprite sheets**, such as `GJ_GameSheet03`. The way one shows these textures is with the `CCSprite` class; it is a simple node that loads a texture and displays it. `CCSprite` is a bit unique in that it has two main `create` functions: `CCSprite::create` for creating nodes out of individual images, and `CCSprite::createWithSpriteFrameName` for creating nodes out of spritesheet images.

It is important to use the correct `create` function for `CCSprite`, as the wrong one will return `nullptr` and cause an error if not properly handled. If the sprite you're creating is contained in its own file, like `GJ_button_01.png`, then you should use `CCSprite::create`; otherwise, if the sprite is in a spritesheet like `GJ_infoIcon_001.png` in `GJ_GameSheet03`, use `CCSprite::createWithSpriteFrameName` instead.

```cpp
// Uh oh! This will return null as GJ_infoIcon_001.png is not its own file but 
// contained in a spritesheet
auto infoSpriteFail = CCSprite::create("GJ_infoIcon_001.png");

// You don't have to specify the name of the spritesheet you're loading from anywhere, 
// as GD has already loaded GJ_GameSheet03 into memory
auto infoSpriteCorrect = CCSprite::createWithSpriteFrameName("GJ_infoIcon_001.png");
```

Even text in Cocos2d is rendered through sprites, or more accurately, **bitmap fonts**. The class for doing this is `CCLabelBMFont`, as shown in one of the above examples. GD's default [Pusab font](https://www.fontsquirrel.com/fonts/pusab) is defined in the `bigFont.fnt` file.

How to add your own sprites to use in Geode mods will be discussed in a later chapter.

## Menus & Buttons

Another important class to know of is `CCMenu`. Most mods need some sort of [buttons](/tutorials/buttons.md) for their UI, and for that, the most common class to use is `CCMenuItemSpriteExtra`. However, all `CCMenuItem`-derived classes **must be part of a `CCMenu` to work**. In practice, this means that all of your buttons must be the direct children of some menu.

You can actually see the effects of this in-game; hold down on some button, and then without releasing move your cursor over other buttons in the same scene. You will find that some buttons display their bounce animation indicating they are usable, and others don't. This is because of the Cocos2d **touch system**, which will be discussed in detail later, but in essence, only buttons in the same menu are clickable when you start holding down from one.

```cpp
auto infoIcon = CCSprite::createWithSpriteFrameName("GJ_infoIcon_001.png");

auto button = CCMenuItemSpriteExtra::create(
    infoIcon,
    // don't worry about these parameters for now
    nullptr, nullptr
);

// The button must be a child of a CCMenu to be usable
auto menu = CCMenu::create();
menu->addChild(button);

// This would make the button visible, but it would not be usable
auto layer = CCLayer::create();
layer->addChild(button);
```

At this point, we're getting very close to writing actual mod code. However, before we can get to that, we must first discuss **GD layers** and the `$modify` macro in Geode.

[Chapter 1.5: Layers](/handbook/vol1/chap1_5.md)

## Notes

> [Note 1] You can also add nodes directly to `CCDirector` and bypass the node tree, but this is very rarely done as nodes in the director don't have access to the touch system or any input for that matter.
