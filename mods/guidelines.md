# Mod Guidelines

These are the guidelines for mods published on the [mods index](https://geode-sdk.org/mods). **All mods on the index are required to follow these rules.**

It is the responsibility of both the mod developer and the [person who verifies the mod](/mods/publishing#who-can-approve-mods) to make sure the mod abides by the rules. If a mod is mistakingly approved, or it receives an update that breaks the rules, or a new rule is added later on, **they may be applied retroactively**. Mods that are found to break the rules due to accidents (or that accidentally start breaking the rules due to external circumstances such as a GD update) may be temporarily delisted until the issues are fixed.

## Everything is Situational

There are hundreds of Geode mods out there, and these guidelines can not possibly cover every single mod that ever has been or will be made. **These guidelines evolve and change over time alongside the modding community, and should at no point be considered a set-in-stone absolute list.** Every rule has exceptions and differing interpretations to it, old rules get removed as attitudes change and new rules get added as clarifications arise.

At the end of the day, **the ultimate power to say what is and isn't allowed on the Index is vested in the Index moderators**\*. If they decide that a mod should or shouldn't be allowed, that's it. Despite the ambiguous wording, **these really are just guidelines, not laws**.

(This rule is referred to with `other-1984`)

> *Realistically speaking, ultimate authority is vested in Lead Developers, and practically speaking whatever HJfod says is usually considered absolute.

## Ban rules

A mod found breaking any of these rules will be rejected unconditionally from the index, as well as very likely leading to a total and permanent ban from posting on the index and a removal of all existing mods on the index.

* (`ban-malware`) The mod contains **malware**. This means any code with the intent purpose of causing harm or any other unwilling negative effects to someone or something.
* (`ban-spyware`) The mod **collects user data without consent**. Collecting data locally for the purpose of statistics is fine, but all sort of online data collection must be unambiguously explicit and able to be opted out of.
* (`ban-stolen`) The mod is **stolen**. This means that it consists fully or nearly fully of the original code of another person with credit explicitly erased.

Please note that a mod purposefully altering the experience of specific users (such as making gameplay unplayable) and mods harrassing specific people and/or their work **is considered malware**, **regardless of the users in question**, unless those users have personally consented to it. This specific form of malware is unlikely to lead to an index-wide ban and will just get the mod delisted, though that depends on the gravity of the situation.

## Rejection rules

> :information_source: As a TL;DR; the rule of thumb for whether something is allowed on the index is **compatibility**. If the mod is or might be incompatible with other mods that it *should* be compatible with, that is usually grounds for a rejection.

Repeated attempts at breaking these rules can result in an **index-wide ban**. Try to make sure that your mod abides by the guidelines. You can contact the index staff if you have any questions about your specific case.

A mod found breaking any of these rules will be **rejected unconditionally** from the index.

 * (`reject-lies`) The mod contains **false information** about what it does.
 * (`reject-ace`) The mod introduces **severe security vulnerabilities** like RCE.
 * (`reject-nsfw`) The mod features **explicit content (NSFW)**. This means gore, obscene violence and/or porn. You are not allowed to put porn on the index. I'm hoping this rule never has to be enforced.
 * (`reject-bad-metadata`) The mod **does not have appropriate metadata**. All mods on the Index must have a proper name, description, icon, and tags, [including jokes](#joke-mods).
 * (`reject-enshittification`) The mod **removes Vanilla features** for no good reason.
 * (`reject-runtime-code`) The mod **installs and runs arbitary code**. You are not allowed to install other mods from outside the Geode index. There are two exceptions to this rule: mods that introduce some sort of scripting system that runs *sandboxed* arbitary code, and mods installing additional paid features as a runtime library. Do note that for the purposes of verification, the code for all of those paid features must be included, including the code used to load them.
 * (`reject-vibecoded`) The mod's **codebase entirely or mostly consists of AI-generated code**. If it appears the programmer does not understand the code that has been generated and is trying to get vibecoded mods on the index, this will result in a rejection. Using AI tools to boost productivity is fine, but fully vibecoding mods will inevitably lead to instability and crashes.
 * (`reject-llm`) The mod **adds an LLM to the game** that does not [adhere to our rules on generative AI](#mods-using-generative-ai).

A mod found breaking any of these rules will likely be rejected, although depending on the situation, they could also still be approved.

 * (`reject-tradmod`) The mod does not use Geode's provided systems for interfacing with GD (such as hooking and patching), resulting in almost guaranteed **mod incompatibility**.
 * (`reject-crash`) The mod **crashes the game** due to [preventible issues](/mods/guidelines-tips#common-pitfalls).
 * (`reject-not-meaningful`) The mod **doesn't do anything meaningful**. Note that joke mods can be considered meaningful; see [our policy on them](#joke-mods).
 * (`reject-hateful`) The mod **contains hateful material**, such as those explained in our policy on [hateful conduct](#hateful-conduct). See also the section of ["Fuck X" mods](#fuck-x-mods).

## Other rules

A mod found breaking any of these rules will likely still be approved, although constant and/or prolific breaking can result in a rejection.

 * (`other-buggy`) The mod has **several common bugs**.
 * (`other-not-saved`) The mod can't **preserve state** between bootups (save data, load data) normally.
 * (`other-illegible`) The mod's **codebase is really hard to read**. The reason this might result in a rejection is if the people verifying the mod for the index are unable to properly ascertain if the mod breaks any of the other rules due to the (hopefully unintended) obfuscation.
 * (`other-incompatible`) The mod **doesn't attempt to be compatible** with other mods. Mods should really try to be compatible with at least mods that do the same features or popular mods that end users are likely to have installed. This can be done in many ways, and at a minimum:
   * Nodes made by the mod *should* have IDs (and if they do, they should be prefixed with the mod id by using _spr: `node->setID("my-fun-node"_spr)`).
   * The mod should also attempt to dodge or at least account for at a minimum Node IDs menus that may be expanded by other mods, such as `bottom-menu` in MenuLayer, but ideally should also account for places where mods might have added buttons.

These are specific edge cases that are explained on their own subsections or pages.

 * (`other-1984`) See [Everything is Situational](#everything-is-situational).
 * (`other-geode-ui`) See [Modifying the Geode UI](#mods-modifying-the-geode-ui).
 * (`other-big-mod`) See ["Too Big to Ignore"](#the-too-big-to-ignore-rule).

## Non-rules

These are some things one might expect to result in a rejection, but actually most likely won't.

 * (`ok-logically-incompatible`) The mod is **logically incompatible** with another mod on the index (for example, two mods that completely reshape `MenuLayer` in different ways are never going to be compatible) - but these mods should be marked as incompatible in the mod's mod.json if they are truly really incompatible.
 * (`ok-ugly`) The mod **looks bad**. We won't shame you for not having HJfod's UI skills! However, if the mod's UI/UX is not just bad but actively harmful, it probably violates the "no false information" rule.
 * (`ok-fucking-swearing`) The mod **contains profanity**. This used to be banned on the index, but as of 2026, profanity is allowed! Do note [the guidelines on "Fuck X" mods](#fuck-x-mods) though.
 * (`ok-language`) The mod is **not in English**. That's just cool! It might take us a while to verify a mod whose source code is written in another language though.

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

(Refer to this guideline with `other-geode-ui`)

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

## Hateful conduct

Developers may be banned off the mods index and other Geode-related online spaces such as the Discord server and the GitHub organization, if they have committed acts in *or outside* the Geode community that are considered hateful, harmful, or otherwise detestable. **These include, but are not limited to:**

 * Stealing or defacing other people's works
 * Engaging in indecent activity with minors
 * Racism, homophobia, transphobia, or any other form of bigotry
 * Any form of public harrassment

This ruling is in place to ensure that Geode stays a **safe and welcoming place for everyone**. This is a mod loader for a children's video game - we do not want to indirectly publicize mean people or their actions with it!

Do take note that **the list above is not comprehensive nor set in stone**, and what qualifies as hateful conduct may and will change and mold over time, with the final say always being left to the Geode team of that time.

Do also note that these rules may very well be enforced retroactively. While we do believe in and hope for people to grow to be better, any person ruled to have committed hateful conduct in the past must give reason for the Geode Team to believe they actually have changed to overturn the verdict.

## "Fuck X" mods

Many mods are often made with the sole purpose of fixing vanilla issues, such as bugs, or one of the many weird quirks of Cocos2d-x. When these mods are then being published on the index, developers sometimes feel the urge to name their mod in relation to the original vanilla feature – often with the tagline "Fuck [Feature Name Here]". This is **strongly discouraged** on the Geode index, and if HJfod has any say on the matter, **straight up banned**.

There are several reasons for this:
 * In nearly every single case, naming the mod "Better [Feature Name Here]" is just objectively a better (heh) name. It's simply more clear.
 * The average user will probably not know what the feature you're ranting about actually is, and they definitely won't know why the feature sucks. I know all of us modders agree touch prio was invented by literally Satan, but for the average GD player, "prio" sounds like a medical term and "touch" is what a suspicious number of top GD players get exposed of doing with kids.
 * Do we really want our community to be defined by RobTop's coding skills? Do we want every mod to act as a constant reminder that the game we're modding has some dodgy pieces of code here and there? Do we want to discount all of the hundreds of thousands of lines of code that, while definitely improvable, still do in fact work and power a game that is played my millions and enjoyed by us to the point we wanted to create the most sophisticated C++ modding framework on the planet just to take this square jumping game to the next level? Is that *really* all we have going for us?

This is of course not limited to just naming mods "Fuck [Feature Name Here]", but that is simply the most common manifestation of this behaviour. This is, of course, not a hard rule that'd ever end up getting someone's mod taken down – it's just a little wish from HJfod for people to calm down.

## The "Too Big to Ignore" Rule

Like everything else in this world, the Geode index also has one set of rules for the ultra rich and another for the rest. Although this is undesirable and we try our best to be fair and equal, some mods are officially considered "too big to ignore" and may be granted small exceptions to index rules that other mods would have to follow. Do note that we do still of course have hard limits; even a mod that is too big to ignore can't go around stealing user data. In addition, "too big to ignore" status also usually just naturally brings with it responsibility for the mod developer to do good; a big mod being caught stealing user data would result in massive outrage immediately. So, for this reason, mods that are too big to ignore may be granted exceptions in some rules, such as not requiring source code to be submitted.

(Refer to this rule as `other-big-mod`)
