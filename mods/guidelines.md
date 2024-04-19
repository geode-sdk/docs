# Mod Guidelines

These are the guidelines for how mods are allowed to the [mods index](https://geode-sdk.org/mods). **All mods on the index are required to follow these rules.**

It is the responsiblity of both the mod developer and the [person who verifies the mod](/mods/publishing#who-can-approve-mods) to make sure the mod abides by the rules. If a mod is mistakingly approved, or it receives an update the breaks the rules, **they may be applied retroactively**.

## Ban rules

A mod found breaking any of these rules will be rejected unconditionally from the index, as well as very likely leading to a total and permanent ban from posting on the index and a removal of all existing mods on the index.

 * The mod contains **malware**. This means any code with the purpose of causing harm to someone or something
 * The mod **collects user data without consent**. Collecting data locally for the purpose of statistics is fine, but all sort of online data collection must be unambiguously explicit and able to be opted out of.
 * The mod is **stolen**. This means that it consists fully or nearly fully of the original code of another person with credit explicitly erased

Please note that a mod purposefully altering the experience of specific users (such as making gameplay unplayable) **is considered malware**, regardless of the users in question. This specific form of malware is unlikely to lead to an index-wide ban, though that depends on the gravity of the situation.

## Hard-rejection rules

A mod found breaking any of these rules will be rejected unconditionally from the index.

 * The mod contains **false information** about what it does or who made it
 * The mod does not use Geode's provided systems for hooking and patching, resulting in almost guaranteed **mod incompatability**
 * The mod introduces **severe security vulnerabilities** like RCE
 * The mod features **explicit content and/or profanity**

## Strong rules

A mod found breaking any of these rules will likely be rejected, although depending on the situation, they could also still be approved.

 * The mod **breaks other mods** for no good reason
 * The mod has several common **game-crashing bugs** that haven't been fixed despite reports
 * The mod **doesn't do anything meaningful**. Jokes, pranks, and test mods are likely to be rejected

## Ordinary rules

A mod found breaking any of these rules will likely still be approved, although constant and/or prolific breaking can result in a rejection.

 * The mod has **several common bugs**
 * The mod can't **preserve state** between bootups (save data, load data) normally

## Non-rules

These are some things one might expect to result in a rejection, but actually most likely won't.

 * The mod is **logically incompatible** with another mod on the index (for example, two mods that completely reshape `MenuLayer` in different ways are never going to be compatible)
 * The mod **looks bad**. We won't shame you for not having HJfod's UI skills

## Mods using Generative AI

Due to controversy among developers regarding the subject, mods that utilize **Generative AI** have to follow these additional guidelines in relation to their AI usage.

 * The mod **may not grant an unfair advantage** in gameplay or anything else to people through the use of AI
 * The mod **may not compromise the integrity of user-created content**. In essence, this means that **no mod is allowed to use AI for placing, modifying, or deleting objects**, though it is not limited to just those.
 * The AI model is only, and as provably as one can reasonably expect, **trained on consentually and transparently obtained data**. Any mod found to be breaking this rule would be found to be in violation of the "collecting user data without consent" hard-ban rule, and subsequently result in the developer being immediately and permanently banned from the index
 * The mod **must visibly and unambiguously communicate to the user its use of AI** prior to them clicking on it (similar to [paid mods](/mods/publishing#what-about-paid-mods)).

In addition, all mods utilizing AI **must be manually verified** on the index and **approved unilaterally by Geode lead developers**, regardless of whether the developer is verified on the index or not.
