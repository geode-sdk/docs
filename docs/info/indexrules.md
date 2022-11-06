# Mods Index Review Guidelines

These are the guidelines for how mods are allowed on the [mods index](https://github.com/geode-sdk/mods). Anyone who with [the power to approve mods](#who-can-approve-mods), **please read this document** and adhere to the guidelines specified. For modders, if you are interested in publishing your mods on the index, you can make the review process faster by **making sure your mod adheres to these guidelines** ahead of time :)

### Hard-rejection rules

A mod found breaking any of these rules will be rejected unconditionally from the index.

 * The mod contains malware. Breaking this guideline will also lead to an immediate and permanent ban from all Geode-related servers and the removal of all of the author's mods from the index

 * The mod contains false information (for example, claims to do something it doesn't, or has false claims about who made it)

 * The mod uses terrible practices that have a near-100% chance of resulting in mod incompatability

   * The mod introduces its own hooking framework

   * The mod modifies the state of the Geode Loader or other mods in a way that breaks them

   * The mod re-implements shared functionality already available through other API mods (custom keybinds, etc.)

 * The mod introduces severe security vulnerabilities like RCE

 * The mod features explicit content (I'm pretty sure this doesn't need to be mentioned, but adding it just in case)

### Strong rules

A mod found breaking any of these rules will likely be rejected, although depending on the situation and severity, they could also still be approved.

 * The mod uses bad practices that have a high chance of resulting in mod incompatability

   * The mod modifies existing GD layers through absolute child indices

 * The mod doesn't work as intended

 * The mod has game-crashing bugs

### Ordinary rules

A mod found breaking any of these can still be approved, although breaking multiple can lead to a rejection.

 * The mod has numerous common non-severe bugs

 * The mod can't preserve state between bootups (save data, load data) normally

 * The mod is incompatible with another mod on the index due to its own fault

### Non-rules

These are some things one might expect to result in a rejection, but actually most likely won't.

 * The mod is fundamentally incompatible with another mod on the index (for example, two mods that completely reshape `MenuLayer` are never going to be compatible)

 * The mod's UX is absolutely god awful

 * The mod doesn't do anything particularly interesting or useful

### Who can approve mods?

Anyone with permission to approve pull requests on the index repository.

**If you want your mod on the index, submit a pull request**, and someone will comment on your PR approving it or pointing out things that cause it to be rejected.

However, if your mod is closed source, you will have to send it source code privately to Geode developers who have permission to add mods to the index.

In general, all Lead Geode Devs can add new mods to the index. However, in practice, for most mods you should ask **HJfod** for mods available on Windows and **alk** or **Camila** for mods on MacOS. For cross-platform mods, ask anyone.

 * HJfod (Windows) | HJfod#1795
 * Pie (Windows) | pie#1593
 * Mat (Windows) | Mat#7533
 * Alk (MacOS) | alk1m123#3076
 * Camila (MacOS) | camila314#4124
