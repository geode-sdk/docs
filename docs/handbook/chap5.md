# Chapter 5: Layers

In [the previous chapter](/docs/handbook/chap3.md), it was mentioned that at all times there exists **one scene at the top of the node tree**. Usually, in practice, this scene has only one child: some **layer**. Scenes are usually only used as containers for layers and for **transitions**; you will likely never have to inherit from `CCScene` in your code. Instead, the class your layers and GD layers usually inherit from is `CCLayer`.

Nearly all layers have their own class; for example, the icon kit is `GJGarageLayer`, the editor is `LevelEditorLayer`, and when you go to play a level you enter `PlayLayer`. The first layer most GD modders start modding and also the first layer in the game (after the loading screen) is **the main menu**; it is an instance of the `MenuLayer` class.

### Finding Layer Names

You can find the **class name** of a given layer using the [DevTools](https://github.com/hjfod/devtools) mod in Geode. Traditionally, the most common mod for inspecting the scene was [CocosExplorer](https://github.com/matcool/CocosExplorer), however as it is a traditional mod, it does not work with Geode.

To see how it works, install the **DevTools** mod in-game through the **Download** tab in Geode and then press **F11** to activate it:

<img src="/imgs/DevTools_MenuLayer.png" alt="Image of the DevTools mod open in GD, focused on MenuLayer" />

!> Note that DevTools is, as of writing (24/10/2022), wholly unfinished and will require quite a bit of manual dragging around to get usable and may crash.

If you look at the **Tree** view on the left, you will see that the current scene contains one layer named **MenuLayer**, which then further contains a bunch of `CCMenu`s and other nodes. The menus contain all of the buttons in the scene; for example, the bottom center row of buttons is contained in one menu.

One thing you will notice in DevTools is that many of the nodes have **string IDs**; this is a **Geode-specific addition** meant to make modding simpler. You can read more about string IDs [in its dedicated article](/docs/tutorials/nodetree.md).

There may also be multiple layers in a scene at once. For example, if you click the profile button in `MenuLayer`, you will find it adds a layer named `ProfilePage` in the scene:

<img src="/imgs/DevTools_ProfilePage.png" alt="Image of the DevTools mod open in GD, showing MenuLayer with ProfilePage on top" />

Using DevTools, you can find the name of any layer. Just navigate to the layer whose name you want to figure out in-game, open up DevTools, and look at the node tree!

?> DevTools also comes with many other utilities, such as moving nodes in the scene around. However, this tutorial is not about DevTools, so you will have to look at its (**currently non-existent**) documentation for that ;)

So now we know that the main menu is called `MenuLayer`, but so what? How can we actually modify it? For that, see [Chapter 6: Modifying Layers](/docs/handbook/chap6.md)
