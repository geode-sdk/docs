---
layout: default
title: Modding Guidelines
parent: Usage
nav_order: 2
description: "Modding Guidelines"
---

# Modding Guidelines

As Geode is a framework meant first and foremost to provide a centralized and simple framework for mods, there are a few rules for modders that we request every modder to follow. These guidelines are meant to ensure compatability and a safe & enjoyable experience for end users.

Failing to comply with these guidelines will result in your mod being denied entry to the `mods` repository. You are still allowed to distribute the mod yourself, however we hope you understand that these rules exist for a reason, and that a mod that harms its users is likely not allowed by law.

Any mod found to be harmful will be publicly reported as such and will be warned against. Its developer may also receive a temporary or permanent ban from the `mods` repository, or in severe cases off all Geode-related platforms.

## 1. Don't be evil

Do not delete System32, do not encrypt their filesystem and ask for a Bitcoin ransom, etc.

### 1.1. Do not collect any data off the user's game / computer without their consent.

This does not apply to data that is collected and stored only locally, so for example making something like a statistics mod that stores the statistics on the user's computer / on their personal Google Drive account is fine. However, also sending yourself a copy of that data / in any other way sending it outside of where the user has agreed to is strictly off limits. Any mod found collecting user data without the user's knowledge / consent will be considered severely harmful.

## 2. Use Geode appropriately

Write reasonable code, basically.

### 2.1. Use Geode's solutions (if they exist)

If Geode has a solution to a problem, use that and do **not** create your own solution to it. If you want to add custom keybinds to your mod, use the Geode API. If you feel the solution Geode provides is insufficient for your needs, please [let us know](/docs/contributing) and we'll look into expanding it.

If Geode comes with a solution for adding nodes to a certain layer, use that instead of hooking the layer yourself.

Do note that "Geode" here includes the loader and the official API mod.

### 2.2. Be careful with open access

Geode is built as an open framework where any mod can access basically any data it needs, but this doesn't mean that they *should* access it. For example, modifying another mod's `Mod` class should only be done in situations where they have intended for that to be done.

### 2.3. Follow Geode's conventions when using it

You should always follow the recommended conventions for naming mod IDs, writing mod descriptions, etc..

## 3. General Modding Guidelines

These are guidelines that could apply to non-Geode mods aswell.

### 3.1. Keep your mod focused

Keep your mod focused on its features and don't try to cram all the ideas you have for the whole of GD into one mod. Try to limit the number of places your mod affects to as small an amount as possible. If you make an icon kit management mod, that mod shouldn't also add a new main level or a Cut button in the Editor.

If you want to bundle a bunch of mods together, consider releasing a modpack instead.

### 3.2. Be as clear with errors as possible

`Insufficient permissions to read file /some/path/to/file` is better than `Can't read file` which is better than `Error Code 15`

### 3.3. Do your best not to crash the game

Never call `autorelease` on classes that inherit from `TableViewCell` and are used with `BoomListView`

