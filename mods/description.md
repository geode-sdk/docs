---
description: How to use about.md in a Geode mod
---

# `about.md`

Geode mods can specify a long, free-form description typeset using Markdown by including a file named `about.md` at the root of their project. See [the MDTextArea class](/classes/geode/MDTextArea) for information on what features of Markdown are supported.

## Example

```md
# My Awesome Mod

This is my awesome mod, that does **all** the awesome stuff!

If you like this mod, please check [my other mod](mod:my.other-mod)!

## Credits

[Join my Discord](https://discord.gg/K9Kuh3hzTC)! Thanks to [Hu Tao](https://www.youtube.com/watch?v=8oap-n_OEgc) for helping with the mod!
```

# `changelog.md`

You may also provide a markdown file that contains a changelog for your mod. In future updates, Geode may use this file to show release notes for the mod.

```md
# v1.1.0

 * Removed Herobrine

# v1.0.1

 * Fixed bugs

# v1.0.0

 * Initial release
```

# `support.md`

You may also provide a markdown file that contains free-form information about how to support you, the mod's developer. This may include things like Patreon links, PayPal links, catgirl picture donation links, anything you wish to say!
