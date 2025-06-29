---
title: Special Markdown Files
description: How to use about.md and other Markdown files in a Geode mod
---

# `about.md`

Geode mods can specify a long, free-form description typeset using Markdown by including a file named `about.md` at the root of their project. See [the MDTextArea class](/classes/geode/MDTextArea) for information on what features of Markdown are supported.

This file is similar to a README, however **it's intended for people who use the mod**; you should also keep a README for developers interested in building / contributing to your mod.

> :information_source: As of [CLI v3.5.0](https://github.com/geode-sdk/cli/releases/tag/v3.5.0), if a `README.md` is present but not an `about.md`, the readme will be used as the mod description. \
> If you don't want this behavior, then make an `about.md` file like usual.


## Example

```md
# My Awesome Mod

This is my awesome mod, that does **all** the awesome stuff!

If you like this mod, please check [my other mod](mod:my.other-mod)!

## Credits

[Join my Discord](https://discord.gg/K9Kuh3hzTC)! Thanks to [Hu Tao](https://www.youtube.com/watch?v=8oap-n_OEgc) for helping with the mod!
```

# `changelog.md`

A markdown file that contains a changelog for your mod. Geode uses this file to show release notes for the mod.

## Example

```md
# v1.1.0

 * Removed Herobrine

# v1.0.1

 * Fixed bugs

# v1.0.0

 * Initial release
```

# `support.md`

A markdown file that contains free-form information about how to support you, the mod's developer. This may include things like links to your Patreon, PayPal, Ko-Fi, etc. - anything you wish to include!

```md
# Thank You for using Rattledasher 5000!

If you like this mod and would like to support me, I have an [OnlyFans](https://globed.dev/soggy/)! You can see me rattle my dashes like a pro if you know what I mean ;)
```
