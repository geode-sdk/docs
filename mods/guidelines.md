# Mod Guidelines

These are the guidelines for mods published on the [mods index](https://geode-sdk.org/mods). **All mods on the index are required to follow these rules.**

It is the responsibility of both the mod developer and the [person who verifies the mod](/mods/publishing#who-can-approve-mods) to make sure the mod abides by the rules. If a mod is mistakingly approved, or it receives an update that breaks the rules, or a new rule is added later on, **they may be applied retroactively**. Mods that are found to break the rules due to accidents (or that accidentally start breaking the rules due to external circumstances such as a GD update) may be temporarily delisted until the issues are fixed.

## Everything is Situational

There are hundreds of Geode mods out there, and these guidelines can not possibly cover every single mod that ever has been or will be made. **These guidelines evolve and change over time alongside the modding community, and should at no point be considered a set-in-stone absolute list.** Every rule has exceptions and differing interpretations to it, old rules get removed as attitudes change and new rules get added as clarifications arise.

At the end of the day, **the ultimate power to say what is and isn't allowed on the Index is vested in the Index moderators**\*. If they decide that a mod should or shouldn't be allowed, that's it. Despite the ambiguous wording, **these really are just guidelines, not laws**.

> *Realistically speaking, ultimate authority is vested in Lead Developers, and practically speaking whatever HJfod says is usually considered absolute

## Ban rules

A mod found breaking any of these rules will be rejected unconditionally from the index, as well as very likely leading to a total and permanent ban from posting on the index and a removal of all existing mods on the index.

 * The mod contains **malware**. This means any code with the intent purpose of causing harm or any other unwilling negative effects to someone or something.
 * The mod **collects user data without consent**. Collecting data locally for the purpose of statistics is fine, but all sort of online data collection must be unambiguously explicit and able to be opted out of.
 * The mod is **stolen**. This means that it consists fully or nearly fully of the original code of another person with credit explicitly erased.

Please note that a mod purposefully altering the experience of specific users (such as making gameplay unplayable) and mods harrassing specific people and/or their work **is considered malware**, **regardless of the users in question**, unless those users have personally consented to it. This specific form of malware is unlikely to lead to an index-wide ban and will just get the mod delisted, though that depends on the gravity of the situation.

## Other rules

> :information_source: As a TL;DR; the rule of thumb for whether something is allowed on the index is **compatibility**. If the mod is or might be incompatible with other mods that it *should* be compatible with, that is usually grounds for a rejection.

Repeated attempts at breaking these rules can result in an **index-wide ban**. Try to make sure that your mod abides by the guidelines. You can contact the index staff if you have any questions about your specific case.

A mod found breaking any of these rules will be **rejected unconditionally** from the index.

 * The mod contains **false information** about what it does
 * The mod does not use Geode's provided systems for interfacing with GD (such as hooking and patching), resulting in almost guaranteed **mod incompatibility**
 * The mod introduces **severe security vulnerabilities** like RCE
 * The mod features **explicit content and/or profanity**. Note that the source code having comments, variable names, etc. containing profanity is fine, as long as they aren't visible to the end user
 * The mod **does not have appropriate metadata**. All mods on the Index must have a proper name, description, icon, and tags, [including jokes](#joke-mods)
 * The mod **removes Vanilla features** for no good reason
 * The mod **installs and runs arbitary code**. You are not allowed to install other mods from outside the Geode index. There are two exceptions to this rule: mods that introduce some sort of scripting system that runs *sandboxed* arbitary code, and mods installing additional paid features as a runtime library. Do note that for the purposes of verification, the code for all of those paid features must be included, including the code used to load them.
 * The mod's **codebase entirely or mostly consists of AI-generated code**. If it appears the programmer does not understand the code that has been generated and is trying to get vibecoded mods on the index, this will result in a rejection. Using AI tools to boost productivity is fine, but fully vibecoding mods will inevitably lead to instability and crashes.

A mod found breaking any of these rules will likely be rejected, although depending on the situation, they could also still be approved.

 * The mod **is incompatible with other mods for no good reason**. Note that mods can be fundamentally incompatible (such as two different texture pack managers)
 * The mod **crashes the game** due to preventible issues
 * The mod **doesn't do anything meaningful**. Note that joke mods can be considered meaningful; see [our policy on them](#joke-mods)

A mod found breaking any of these rules will likely still be approved, although constant and/or prolific breaking can result in a rejection.

 * The mod has **several common bugs**
 * The mod can't **preserve state** between bootups (save data, load data) normally
 * The mod's **codebase is really hard to read**. The reason this might result in a rejection is if the people verifying the mod for the index are unable to properly ascertain if the mod breaks any of the other rules due to the (hopefully unintended) obfuscation

## Non-rules

These are some things one might expect to result in a rejection, but actually most likely won't.

 * The mod is **logically incompatible** with another mod on the index (for example, two mods that completely reshape `MenuLayer` in different ways are never going to be compatible)
 * The mod **looks bad**. We won't shame you for not having HJfod's UI skills! However, if the mod's UI/UX is not just bad but actively harmful, it probably violates the "no false information" rule
 * The mod is **not in English**. That's just cool! It might take us a while to verify a mod whose source code is written in another language though

There are also some **code patterns** that one might assume would be rejected, but are actually allowed:

```cpp
#ifdef GEODE_IS_WINDOWS
auto editorUIGridSize = reinterpret_cast<float*>(reinterpret_cast<uintptr_t>(EditorUI::get()) + 0x1d0);
#endif
```
Accessing & modifying members by their raw platform offset is totally okay, as this does not pose any interoperability concerns with other mods, it just makes your mod harder to port.

```cpp
if (this->querySelector("hjfod.gdshare/export-button")) {
    // GDShare is loaded, compatibility code
}
```
Checking that another mod is loaded by looking for identifying effects of them in the game is completely fine, as long as the checks are safe and don't make assumptions about things that haven't been checked. For example, just doing `Loader::get()->isModLoaded("hjfod.gdshare")` is NOT enough to guarantee all of GDShare's buttons exist, so you should always check for the buttons themselves!

## Mods modifying the Geode UI

Some mods may want to **extend Geode's own UI**, for example to add custom features to their own mod's page. This is allowed, however with some caveats. See [the tutorial page](/tutorials/modify-geode) for more.

## Joke mods

Joke mods, as in mods whose whole function is to just be a funny little goof and nothing else, **are allowed**, as long as they have a proper name, description, icon, and tags, particularly the `Joke` tag.

However, do note that for a mod to be listed on the Index it has to be **meaningful**. That is, it needs to be something that people might actually want to have and use. The precise definition is up to whoever is approving the mod, but in general **if a mod is something you would install once for five minutes then uninstall and never use again, it's probably not meaningful**. This mostly includes mods that actively make playing the game worse without adding new kinds of fun, but it can also include visual tweaks so small and unintrusive you forget you even have it installed.

For example:

 * A mod that just changes the title of the game to Geometry Fart and nothing else is probably not enough to be accepted
 * A mod that deliberately crashes the game on every death is very unlikely to be accepted, as that is something which is funny for one YouTube video but so actively harmful no one would use it normally
 * A mod that changes physics to super weird ones might be enough to be accepted, as while most users won't have it permanently enabled, they may enable it every now and then to use together with other mods like Globed
 * A mod that turns the player icon into a badly drawn Sonic the Hedgehog is probably enough to be accepted
 * A mod that is a compilation of multiple different toggleable small jokes is absolutely enough to be accepted, provided it doesn't break any other rules

## Paid mods

Paid mods, aka mods that require payment to be used, **are allowed on the index as long as they have the `Paid` tag**. Do however note that Geode does not handle any payments - you will need to set up all of the payment infrastructure, DRM, etc. by yourself.

Mods that are usable for free but have extra integrated paid features, such as a Pro version or a Supporter tier, do not need to include the `Paid` tag unless the paid features are essential to the functioning of the mod. For example, a multiplayer mod that has a paid tier for private rooms but free public rooms for everyone would likely not need the tag.

It is also okay to publish an **installer mod** for a paid mod on the index, aka something that just asks the user to authenticate and then installs the paid content. These mods also need to have the `Paid` tag.

## Mods based on other people's mods

Sometimes rather than making original ideas, modders want to for one reason or another make a mod based on someone else's mod, be that for porting it or for making an enhanced version of it. This is allowed under the following conditions:

 * The original creator of the mod **must approve of it**. Stealing is never allowed!
 * The new mod must have some **meaningful changes** compared to the original, for example major bugfixes or a port to a new GD version. Simply repackaging Click Between Frames as Superb Input Precision is not allowed
 * The original mod **must be credited**, as per the no stealing rule

## Mods using Generative AI

> :information_source: Note that mods *coded* using Generative AI are subject to [different rules](#other-rules); this section is about mods that bring Generative AI into the game itself

Due to controversy among developers regarding the subject, mods that utilize **Generative AI** have to follow these additional guidelines in relation to their AI usage.

> :information_source: TL;DR; All mods using AI must be trained on consentually obtained data, and may not be used for creating levels.

 * The mod **may not grant an unfair advantage** in gameplay or anything else to people through the use of AI
 * The mod **may not compromise the integrity of user-created content**. In essence, this means that **no mod is allowed to use AI for placing, modifying, or deleting objects**, though it is not limited to just those.
 * The AI model is only, and as provably as one can reasonably expect, **trained on consentually and transparently obtained data**. Any mod found to be breaking this rule would be found to be in violation of the "collecting user data without consent" hard-ban rule, and subsequently result in the developer being immediately and permanently banned from the index
 * The mod **must visibly and unambiguously communicate to the user its use of AI** prior to them clicking on it. This can be as simple as adding "AI" or "GPT" to the name of the mod.

In addition, all mods utilizing AI **must be manually verified** on the index and **approved unilaterally by Geode lead developers**, regardless of whether the developer is verified on the index or not.

## Possible pitfalls

While nothing in this section are absolute rules, they are red flags that index staff look for while reviewing a mod. These pitfalls describe common issues that may cause crashes or incompatibility with other mods/the game.

* Avoid generating exceptions.

Due to compatibility issues, functions that generate C++ exceptions should be avoided, _even if you catch the exception_. Either verify that the exception will not be generated beforehand, or use an alternative function that does not throw one. One such function is `std::stoi` and `std::stof`, which should be replaced with Geode's [`numFromString`](/functions/geode/utils/numFromString/).

Functions from the `<filesystem>` header should be handled carefully. Most filesystem functions have `std::error_code` overloads, which do not throw exceptions on failure. Use them. Calling `string()` on a `std::filesystem::path` must also be avoided - this may crash on certain setups. Instead, use [`pathToString`](/functions/geode/utils/string/pathToString/).

Calling `unwrap()` on a [Result](/classes/geode/Result/) without first checking if it is `ok()` will lead to an instant rejection. You can never be certain that your web calls will always succeed!

* Avoid usage of `dynamic_cast`.

Due to how dynamic casting is implemented, usage of this function will usually fail for classes from other mods and the game. Instead, use [`typeinfo_cast`](/functions/geode/cast/typeinfo_cast/).

* Avoid blocking the main thread.

While it is unavoidable in some cases (or will cause additional instability), offloading expensive functions to a separate thread avoids the dreaded "application not responding" and greatly improves the user experience. [Tasks](/tutorials/tasks) were designed for this use case and can be used to wrap any callback-based async api. Mods blocking the main thread for web requests will be instantly rejected - please use Geode's [web utils](/tutorials/fetch).

* Avoid reinventing the wheel.

Hooking functions, especially heavily called ones, to accomplish tasks that can be done by other means may get your mod rejected. One example would be hooking `CCScheduler::update` to run code on each frame (use [`scheduleSelector`](/classes/cocos2d/CCScheduler#scheduleSelector) instead) or to keep a node across scenes (use [`SceneManager::keepAcrossScenes`](/classes/geode/SceneManager#keepAcrossScenes)). Besides creating performance issues, this may also reduce compatibility with other mods.

While this isn't typically a rejection reason, new developers tend to find themselves "reinventing the wheel" and using platform-specific functions to accomplish tasks that are already covered by Geode utils. For example, [restarting the game](/functions/geode/utils/game/restart/), [opening a file picker](/functions/geode/utils/file/pick) and [splitting a string](/functions/geode/utils/string/split) are all built into the Geode utils. These functions have the most compatibility with other mods and are tested across all supported platforms. A basic list of them can be found [here](/tutorials/utils).

Developers should also be aware of Geode's [directory functions](https://github.com/geode-sdk/geode/blob/main/loader/include/Geode/loader/Dirs.hpp), which can be used for storing mod files and determining the location of game data. Unless you have good reason, mod files should be stored within the mod's save directory, which can be determined with [`Mod::getSaveDir`](/classes/geode/Mod#getSaveDir) (using the config directory through `getConfigDir` is also fine).

* Avoid submitting locally built mods.

For new developers, we strongly recommend making use of the cross-platform [build-geode-mod](https://github.com/geode-sdk/build-geode-mod/) action that comes with the default mod template. This action helpfully builds your mod for all platforms, which makes it very easy for developers to support them. 

Those who are uploading their code on platforms outside of GitHub should also be using the CI instances offered by those platforms for building their mod. This makes it easier for us to verify that a mod has not been tampered with.

## Hateful conduct

Developers may be banned off the mods index and other Geode-related online spaces such as the Discord server and the GitHub organization, if they have committed acts in *or outside* the Geode community that are considered hateful, harmful, or otherwise detestable. **These include, but are not limited to:**

 * Stealing or defacing other people's works
 * Engaging in indecent activity with minors
 * Racism, homophobia, transphobia, or any other form of bigotry
 * Any form of public harrassment

This ruling is in place to ensure that Geode stays a **safe and welcoming place for everyone**. This is a mod loader for a children's video game - we do not want to indirectly publicize mean people or their actions with it!

Do take note that **the list above is not comprehensive nor set in stone**, and what qualifies as hateful conduct may and will change and mold over time, with the final say always being left to the Geode team of that time.

Do also note that these rules may very well be enforced retroactively. While we do believe in and hope for people to grow to be better, any person ruled to have committed hateful conduct in the past must give reason for the Geode Team to believe they actually have changed to overturn the verdict.

## The "Too Big to Ignore" Rule

Like everything else in this world, the Geode index also does have one set of rules for the ultra rich and another for the rest. Although this is undesirable and we try our best to be fair and equal, some mods are officially considered "too big to ignore" and may be granted small exceptions to index rules that other mods would have to follow. Do note that we do still of course have hard limits; even a mod that is too big to ignore can't go around stealing user data. In addition, "too big to ignore" status also usually just naturally brings with it responsibility for the mod developer to do good; a big mod being caught stealing user data would result in massive outrage immediately. So, for this reason, mods that are too big to ignore may be granted exceptions in some rules, such as not requiring source code to be submitted.
