---
description: How to configure Geode mods using mod.json
---

# Configuring mods

Geode mods are configured through a file called `mod.json`, located at the root of your project. The file contains general information about the mod, such as the name, ID, version, and also things like settings, resources, and dependencies.

If you're using VS Code, you should [install the Geode VS Code extension](https://marketplace.visualstudio.com/items?itemName=GeodeSDK.geode), as it provides automatic checking for `mod.json` and intellisense for what keys are available.

## Required settings

There are 6 required properties for every mod: the target Geode version, the ID and name of the mod, its developer, and its version.

```json
{
    "geode": "v1.0.0",
    "gd": {
        "win": "2.206",
        "android": "2.206"
    }
    "id": "my.awesome-mod",
    "name": "My Awesome Mod",
    "version": "v1.0.0",
    "developer": "Me"
}
```
You can also use the `developers` property instead of `developer`

```json
{
    "geode": "v1.0.0",
    "gd": {
        "win": "2.206",
        "android": "2.206"
    }
    "id": "my.awesome-mod",
    "name": "My Awesome Mod",
    "version": "v1.0.0",
    "developers": ["Me","Friendly Developer"]
}
```

## List of keys

### `geode`

The target Geode version. Should be in the format of an exact version, such as `v1.0.0` or `v1.2.0`. The target version should always be the exact version of Geode you are developing with.

### `gd`

The target Geometry Dash version exactly, or `*` for **any** GD version (whether your mod actually works on any GD version is your responsibility.) 

This key is an object for specifying per platform GD version:
```json
{
    "gd": {
        "android": "2.200",
        "win": "2.204"
    }
}
```
The valid platform keys are `win`, `mac`, `android` and `ios`.
```json
{
    "gd": {
        "android": "*" // Mod works on any android gd version
    }
}
```


### `id`

The ID of the mod. May only contain lowercase ASCII characters (`a-z`), periods (`.`), underscores (`_`) and dashes (`-`). Otherwise may be formatted in any way you like, but it is common to make the ID be in the form of `developer.mod-name`. The ID is used to uniquely identify the mod in things such as folder names, URLs, and save data.

### `name`

The name of the mod that is displayed in-game, and in other situations where the mod is displayed. May be any UTF-8 valid string, however do note that GD does not normally support many characters beyond the ASCII character set, so unless the user has a mod or texture pack that extends the character set, not all characters in the name may be displayed.

### `version`

The version of the mod; should follow [semver](https://semver.org), especially if the mod can be used as a dependency (see [the API key](#api)).

### `developer`

The name of the mod's developer, displayed on the Installed tab. Should be a single name, like "HJfod" or "Alk". If the mod has multiple developers, this should be a team name like "Geode Team".

### `developers` 

The name of the mod's developers. Replaces `developer`. Can be a single name, like \["HJfod"\] or \["Alk"\]. If the mod has multiple developers, you should list the names of each developer because, the `developers` property is a list/array. First developer listed is the main developer. If you have 3 or more developers listed, it will show `main developer + 2 more`. If you have 2 developers listed, it will show `Developer 1 & Developer 2`. If you list multiple developers, when someone clicks on your name(s), it will show the list of the developers, then you can click on one to see other mods they have made or are apart of.

### `description`

A short description of the mod. Should only be a single sentence; for longer descriptions, see [about.md](/mods/md-files.md).

### `links`

This key is an object for various links to socials.

* `community` - The link to the mod's Discord server or another community hub.
* `source` - The link to the source code, usually a GitHub repository.
* `homepage` - Your mod's website.

### `issues`

Describes where users can report problems with the mod. Value is an object with the following properties:

 * `info` - Free-form description of where / whom to report issues to

 * `url` - URL to a Discord server, GitHub repository etc. where the issues are reported

### `dependencies`

The dependencies of a mod; see [Dependencies](/mods/dependencies.md) for details

### `incompatibilities`

The incompatibilities of a mod. Very similar to [dependencies](/mods/dependencies.md) but the valid importances are `breaking`, `conflicting` and `superseded`.

* `breaking` - prevents the incompatible mod from loading
* `conflicting` - both mods load anyway but it shows a warning in mod list
* `superseded` - only used in very special circumstances when your mod is meant to show up as an update to a different mod

### `settings`

The settings of a mod; see [Settings](/mods/settings.md) for details

### `resources`

The resources of a mod; see [Resources](/mods/resources.md) for details

### `early-load`

If true, specifies that this mod must have finished loading before the loading screen appears in GD. For example, [TextureLdr](https://github.com/geode-sdk/textureldr) uses this to apply texture packs at startup.

### `api`

Specifies that this mod can be [used as a dependency](/mods/dependencies.md). Value is an object with the following properties:

#### `headers`

An array specifying the list of headers that should bundled with this mod. Supports [globbing](/mods/resources.md).

### `tags`

A list of tags to categorize the mod. A mod can have any amount of tags, but between 1-4 is the recommended amount.

| Tag name | Description |
|----------|-------------|
| `universal` | The mod affects the entire game |
| `gameplay` | The mod affects mainly gameplay |
| `editor` | The mod affects mainly the editor |
| `offline` | The mod does not require an internet connection to work |
| `online` | The mod requires an internet connection to work |
| `enhancement` | The mod enhances (adds more) to an existing GD feature |
| `music` | The mod deals with music, such as adding more songs |
| `interface` | The mod modifies the GD UI in notable ways (beyond just adding a new button) |
| `bugfix` | The mod fixes existing bugs in the game |
| `utility` | The mod provides tools that simplify working with the game and its levels |
| `performance` | The mod optimizes existing GD features |
| `customization` | The mod adds new customization options to existing GD features |
| `content` | The mod adds new content (new levels, gamemodes, etc.) |
| `developer` | The mod is intended for mod developers only |
| `cheat` | The mod adds cheats like noclip |
| `paid` | The mod contains paid content, like a Pro tier, or if the mod acts as an installer for a fully paywalled mod |
| `joke` | The mod is a joke. See [the docs](/mods/guidelines#joke-mods) for what joke mods are considered to be "meaningful" |
