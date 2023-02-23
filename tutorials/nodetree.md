# Getting nodes

There comes a time in every GD modder's life when they are faced with the same eternal question: how to get a specific node from a certain layer. The easiest, and **incontestably best** way to do this is through **member variable access**. If the desired node is stored as a member in the layer, for example `m_playtestBtn` in `EditorUI`, you should __always__ access it through that.

However, this is far from always possible, as not all nodes are stored as members in their containing layers. This poses a problem, the easiest solution to which has been the source of **hours of pain and misery**. The solution referred to here is **getting nodes by their absolute index**.

It should be stressed here, at the very start, that **you should __NEVER__ get nodes by their index unless ABSOLUTELY __necessary__**. It should always be left as a last resort. The Cocos2d node tree is _not_ a stabile or predictable environment; child indexes can change rapidly due to game features, Z-order reordering, or most commonly due to other mods. If you do `node->getChildren()->objectAtIndex(5)`, you have no guarantees of getting the same node every time.

Using absolute indexes also requires a lot of work, as the **indexes are shuffled around** after a layer's `init` function due to Z-order reordering. This means that if you look at a node through [CocosExplorer](https://github.com/matcool/CocosExplorer) or [DevTools](https://github.com/geode-sdk/devtools) and see that is is at index 5, it is more than likely that during the layer's `init` function (where you will most likely be working with the node) this will not be the case.

On top of this, in old GD modding frameworks, there were unfortunately basically no alternatives to members and getting by index. There are node **tags**, however tags are mostly used to implement differing functionality for buttons that use the same callback. The result of this was a lot of mods using raw indexes to position their own additions to GD's UI. Surprise surprise, this lead to [even mods by the same developer messing up their button positions due to indexes being different than expected](https://discord.com/channels/822510988409831486/858820729234391063/881436739250585610). Using raw indexes are one of the biggest sources of **mod incompatability**; and as Geode is meant to help with solving that, it introduces a whole new way of getting nodes:

## String IDs

String IDs are a **Geode-specific addition** to the `CCNode` class that come in three main functions: `CCNode::getID`, `CCNode::setID` and `CCNode::getChildByID`. In all their simplicity, it means that you can assign any `CCNode` a string ID, and get a child by its string ID. On the surface, this may seem quite trivial, but this is actually an **incredibly powerful alternative** to using raw indexes. Whereas raw indexes are shuffled around and unreliable, you can be **almost certain** that `node->getChildByID("specific-node-id")` will **always** return the same node.

For example, using string IDs, adding a new button to the left of the icon kit button in the main menu is as simple as this:
```cpp
class $modify(MenuLayer) {
    bool init() {
        auto menu = this->getChildByID("main-menu");
        auto iconKitBtn = menu->getChildByID("icon-kit-button");

        auto btn = /* create button */;
        btn->setPosition(iconKitBtn->getPosition() - CCPoint { -50.f, 0.f });
        menu->addChild(btn);
    }
};
```
Notice that there are no hardcoded indexes. Even if `main-menu` is at a completely different index than expected, as long as the main menu in `MenuLayer` has the ID `main-menu`, the code will always find it and work as expected.

Although, this code does also have another interoperability concern: sure, we can be sure we added our node to the right place, but what if some other mod does the same? If another mod adds a button to the left of the icon kit button, they would overlap. To solve this issue, Geode introduces [layouts](/tutorials/layouts.md), which are discussed in their own tutorial.

This also highlights the **problem with string IDs**: **someone has to assign them**. Whereas raw indexes are an intrinsic property of all CCNodes, the default ID of a node is an empty string (`""`). Geode does come with [default IDs for common classes like `MenuLayer`](https://github.com/geode-sdk/geode/blob/91cecf3843d246939be4057cdf8e7d5d607aeeb1/loader/src/hooks/MenuLayer.cpp#L150-L197), but there are unfortunately too many layers in GD to label all of them.

This has, however, been taken in mind when designing the string ID system. If you find a layer you want to get a child of is missing string IDs, what you can do is **add them yourself** (using members or raw indexes), and either [send your code for adding them to Geode](https://github.com/geode-sdk/geode/pulls/new) or [let others know your mod adds these IDs](https://discord.gg/9e43WMKzhp). This way, someone else who wants to get the same children from the same layer can easily piggyback off of your logic. While the string ID system is ultimately and unfortunately based on raw indexes, its point is to **provide a single source of truth for those indexes**; once someone has done the work, no one should have to do it again.

## Viewing String IDs

You can see if a layer's nodes have string IDs using the [DevTools](https://github.com/geode-sdk/devtools) mod, which shows them in the **Tree** view.

## String IDs in your own layers

It is recommended for you to also use string IDs in your own layers, as this makes mod-modifying-mod interoperability much easier.

## Naming String IDs

String IDs you give to nodes should be in kebab case with only lowercase a-z letters and no spaces.



