---
description: How to include resources in Geode mods
---

# Resources

Geode mods may include any number of resources/assets in the mod package. Geode also provides utilities for automatically **generating spritesheets**, **turning TTF fonts to Bitmap ones**, and **creating Medium/Low versions of sprites**.

In order for Geode to create the resources, [you must have Geode CLI installed](/geode/installcli).

## Adding sprites

Adding sprites is as simple as adding the PNG files somewhere in your mod's source directory (commonly under `resources`) and then adding them to your [mod.json](/mods/configuring.md).

Geometry Dash needs all sprites to have a High, Medium, and Low quality version for performance. **Geode assumes that the source sprites you provide are UHD quality, and creates the medium / low versions automatically.** You should only include the high quality version of your sprite in resources.

Example for adding sprites:

```json
{
    "resources": {
        "sprites": [
            "resources/MySprite.png",
            "resources/other-sprite.png"
        ]
    }
}
```

You can now use the sprite in-game by using its name, just like any other sprite. Note that **Geode automatically prefixes all sprites to avoid name collisions**, so the actual name of the sprite in code is `my.mod-id/MySprite.png`. For convenience, Geode provides a `_spr` string literal extension that automatically adds this prefix:

```cpp
auto spr = CCSprite::create("MySprite.png"_spr);
```

Additionally, Geode supports **globbing**, so you can include all of the PNGs from a folder at once:

```json
{
    "resources": {
        "sprites": [
            "resources/*.png"
        ]
    }
}
```

## Adding spritesheets

Spritesheets are **better for performance** than ordinary sprites, so it's recommended to put as many of your sprites in a spritesheet as you can. Geode requires all spritesheets to have names; the name should be written in [PascalCase](https://techterms.com/definition/pascalcase).

```json
{
    "resources": {
        "spritesheets": {
            "MySheet": [
                "resources/sheet/*.png"
            ],
            "OtherSheet": [
                "resources/other/*.png"
            ]
        }
    }
}
```

After adding the sheet and its sprites to your `mod.json`, you can now use the sprite in code as normal:

```cpp
auto spr = CCSprite::createWithSpriteFrameName("mySpriteInASheet.png"_spr);
```

## Adding fonts

Geode can automatically convert TTF and OTF fonts into GD-compatible Bitmap fonts. You can add fonts by adding them under the `fonts` key in `resources`:

```json
{
    "resources": {
        "fonts": {
            "MyFont": {
                "path": "resources/myFont.ttf",
                "size": 80
            },
            "OtherFont": {
                "path": "resources/other.ttf",
                "size": 64,
                "charset": "32-500,8226"
            }
        }
    }
}
```

The same `_spr` suffix for sprites can be used for fonts aswell:

```cpp
auto label = CCLabelBMFont::create("Hi mom!", "MyFont.fnt"_spr);
```

The `path` key in a font is a path to the font file. The `size` key is the size of the rendered font characters; this may be any positive number, and a reasonable value for it is whatever is large enough to look good in-game. If the size is too small, the font may look blurry. The larger the size, the larger the font though.

The `charset` key specifies the Unicode codepoints of which characters should be included in the font. GD fonts include `32-126,8226`, which are ASCII letters with a few punctuation marks included. Ranges can be specified with a dash and codepoints should be separated with a comma.

There is also an experimental key `outline` that adds a black outline around the font characters and a small shadow, to make it similar to other GD fonts. As this feature is experimental, be aware that the output quality may vary.

## Adding audio & other files

Any other files can be included through the `files` key:

```json
{
    "resources": {
        "files": [
            "resources/audio/trollface.ogg",
            "resources/scripts/*.js"
        ]
    }
}
```
