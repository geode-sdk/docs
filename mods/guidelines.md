# Mod Guidelines

These are the guidelines for how mods are allowed to the [mods index](https://geode-sdk.org/mods). **All mods on the index are required to follow these rules.**

It is the responsiblity of both the mod developer and the [person who verifies the mod](/mods/publishing#who-can-approve-mods) to make sure the mod abides by the rules. If a mod is mistakingly approved, or it receives an update the breaks the rules, **they may be applied retroactively**.

## Ban rules

A mod found breaking any of these rules will be rejected unconditionally from the index, as well as very likely leading to a total and permanent ban from posting on the index and a removal of all existing mods on the index.

 * The mod contains **malware**.
 * The mod is **stolen**. This means that it consists fully or nearly fully of the original code of another person with credit explicitly erased.

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
