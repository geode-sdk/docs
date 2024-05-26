# Mod Guidelines

These are the guidelines for how mods are allowed to the [mods index](https://geode-sdk.org/mods). **All mods on the index are required to follow these rules.**

It is the responsiblity of both the mod developer and the [person who verifies the mod](/mods/publishing#who-can-approve-mods) to make sure the mod abides by the rules. If a mod is mistakingly approved, or it receives an update the breaks the rules, **they may be applied retroactively**.

## Ban rules

A mod found breaking any of these rules will be rejected unconditionally from the index, as well as very likely leading to a total and permanent ban from posting on the index and a removal of all existing mods on the index.

 * The mod contains **malware**. This means any code with the purpose of causing harm to someone or something
 * The mod **collects user data without consent**. Collecting data locally for the purpose of statistics is fine, but all sort of online data collection must be unambiguously explicit and able to be opted out of.
 * The mod is **stolen**. This means that it consists fully or nearly fully of the original code of another person with credit explicitly erased

Please note that a mod purposefully altering the experience of specific users (such as making gameplay unplayable) **is considered malware**, regardless of the users in question. This specific form of malware is unlikely to lead to an index-wide ban, though that depends on the gravity of the situation.

## Other rules

> :information_source: As a TL;DR; the rule of thumb for whether something is allowed on the index is **compatability**. If the mod is or might be incompatible with other mods that it *should* be compatible with, that is usually grounds for a rejection.

A mod found breaking any of these rules will be rejected unconditionally from the index.

 * The mod contains **false information** about what it does
 * The mod does not use Geode's provided systems for hooking and patching, resulting in almost guaranteed **mod incompatability**
 * The mod introduces **severe security vulnerabilities** like RCE
 * The mod features **explicit content and/or profanity**. Note that the source code having comments, variable names, etc. containing profanity is fine, as long as they aren't visible to the end user

A mod found breaking any of these rules will likely be rejected, although depending on the situation, they could also still be approved.

 * The mod **breaks other mods** for no good reason
 * The mod has several common **game-crashing bugs** that haven't been fixed despite reports
 * The mod **doesn't do anything meaningful**. Jokes, pranks, and test mods are likely to be rejected

A mod found breaking any of these rules will likely still be approved, although constant and/or prolific breaking can result in a rejection.

 * The mod has **several common bugs**
 * The mod can't **preserve state** between bootups (save data, load data) normally
 * The mod's **codebase is really hard to read**. The reason this might result in a rejection is if the people verifying the mod for the index are unable to properly ascertain if the mod breaks any of the other rules due to the (hopefully unintended) obfuscation

## Non-rules

These are some things one might expect to result in a rejection, but actually most likely won't.

 * The mod is **logically incompatible** with another mod on the index (for example, two mods that completely reshape `MenuLayer` in different ways are never going to be compatible)
 * The mod **looks bad**. We won't shame you for not having HJfod's UI skills
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
    // GDShare is loaded, compatability code
}
```
Checking that another mod is loaded by looking for identifying effects of them in the game is completely fine, as long as the checks are safe and don't make assumptions about things that haven't been checked. For example, just doing `Loader::get()->isModLoaded("hjfod.gdshare")` is NOT enough to guarantee all of GDShare's buttons exist, so you should always check for the buttons themselves!

## Mods modifying the Geode UI

Some mods may want to **extend Geode's own UI**, for example to add custom features to their own mod's page. This is allowed, however on the following conditions:

1. The mod may **only access nodes by ID**. No matching types or indices, if the node the mod wants to modify doesn't have an ID, the mod can not edit it
2. The mod must **always check for null** on every node they access. It should never assume the existence of a node, even if it's trivial
3. The mod must **safely handle all possible failure states**, such as missing sprites
4. If the mod finds any missing IDs, it must **undo any/all of its changes** and not make any more. The recommended way to do this is to first use `querySelector` to grab all the nodes in the scene it intends to modify beforehand, and then returning early if any of them are null

These rules are in place because the Geode UI is a highly volatile place that **may change at any time**, and any mod causing the Geode UI itself to crash would make it impossible to disable without safe mode.

## Mods using Generative AI

Due to controversy among developers regarding the subject, mods that utilize **Generative AI** have to follow these additional guidelines in relation to their AI usage.

> :information_source: TL;DR; All mods using AI must be trained on consentually obtained data, and may not be used for creating levels.

 * The mod **may not grant an unfair advantage** in gameplay or anything else to people through the use of AI
 * The mod **may not compromise the integrity of user-created content**. In essence, this means that **no mod is allowed to use AI for placing, modifying, or deleting objects**, though it is not limited to just those.
 * The AI model is only, and as provably as one can reasonably expect, **trained on consentually and transparently obtained data**. Any mod found to be breaking this rule would be found to be in violation of the "collecting user data without consent" hard-ban rule, and subsequently result in the developer being immediately and permanently banned from the index
 * The mod **must visibly and unambiguously communicate to the user its use of AI** prior to them clicking on it (similar to [paid mods](/mods/publishing#what-about-paid-mods)).

In addition, all mods utilizing AI **must be manually verified** on the index and **approved unilaterally by Geode lead developers**, regardless of whether the developer is verified on the index or not.

## Hateful conduct

Developers may be banned off the mods index and other Geode-related online spaces such as the Discord server and the GitHub organization, if they have committed acts in *or outside* the Geode community that are considered hateful, harmful, or otherwise detestable. **These include, but are not limited to:**

 * Stealing or defacing other people's works
 * Engaging in indecent activity with minors
 * Racism, homophobia, transphobia, or any other form of bigotry
 * Any form of public harrassment

This ruling is in place to ensure that Geode stays a **safe and welcoming place for everyone**. This is a mod loader for a children's video game - we do not want to indirectly publicize mean people or their actions with it!

Do take note that **the list above is not comprehensive nor set in stone**, and what qualifies as hateful conduct may and will change and mold over time, with the final say always being left to the Geode team of that time.

Do also note that these rules may very well be enforced retroactively. While we do believe in and hope for people to grow to be better, any person ruled to have committed hateful conduct in the past must give reason for the Geode Team to believe they actually have changed to overturn the verdict.
