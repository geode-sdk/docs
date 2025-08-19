---
title: Tags
order: 1
---

# Tags

In Geometry Dash, there are tags you can use for things such as `FLAlertLayer`, `DialogLayer`, or `MDPopup`...

Let's learn how to use these tags:

## Color

In Geometry Dash, there are color tags, which change the color of text. Below is an example using some of these color tags:

| **Tag** | **Color Code** | **Example**                           |
|---------|----------------|---------------------------------------|
| `<cb>`  | `0x4A52E1`     | <p style="color: #4A52E1;">Sample</p> |
| `<cg>`  | `0x40E348`     | <p style="color: #40E348;">Sample</p> |
| `<cl>`  | `0x60ABEF`     | <p style="color: #60ABEF;">Sample</p> |
| `<cj>`  | `0x32C8FF`     | <p style="color: #32C8FF;">Sample</p> |
| `<cy>`  | `0xFFFF00`     | <p style="color: #FFFF00;">Sample</p> |
| `<co>`  | `0xFF5A4B`     | <p style="color: #FF5A4B;">Sample</p> |
| `<cr>`  | `0xFF5A5A`     | <p style="color: #FF5A5A;">Sample</p> |
| `<cp>`  | `0xFF00FF`     | <p style="color: #FF00FF;">Sample</p> |
| `<ca>`  | `0x9632FF`     | <p style="color: #9632FF;">Sample</p> |
| `<cd>`  | `0xFF96FF`     | <p style="color: #FF96FF;">Sample</p> |
| `<cc>`  | `0xFFFF96`     | <p style="color: #FFFF96;">Sample</p> |
| `<cf>`  | `0x96FFFF`     | <p style="color: #96FFFF;">Sample</p> |
| `<cs>`  | `0xFFDC41`     | <p style="color: #FFDC41;">Sample</p> |
| `<c_>`  | `0xFF0000`     | <p style="color: #FF0000;">Sample</p> |

To use these tags, follow the syntax below:

```
"Welcome to my red <cr>lair</c>! As you can see, my <cg>things</c> are green, and my <cb>doorframe</c> is blue! Why? That's up to <co>Rob</c><cj>Top</c> to answer!"
```

By using `</c>`, you are closing the tag. `</c>` closes all color tags.

## Delay

In Geometry Dash, there are also delay tags to wait a certain amount of centiseconds (1/100 of a second) before more text appearing. Below is an example:

```
Wait!<d040> You shouldn't go there<d500>.<d500>.<d500>.
```

In the `<d...>` tag, the number next to the `<d` specifies the number of centiseconds to wait. However, this number must be 3 digits. This is how it looks like in-game (GIF from GDDocs):

![GIF showing delay tag](/assets/delay_tag.gif)

## Shake

Shake tags make text shake. Below is an example:

```
<co>I HAVE SAID TOO MUCH! QUICKLY TO THE <s260>CHOPPER!</s></c>
```

In this example, the syntax for the shake tag is similar to the syntax of the delay tag. This is however based on the intensity of the shaking. For reference, this is how it looks like in-game (GIF from GDDocs):

![GIF showing shake tag](/assets/shake_tag.gif)

# Instant/Fade

Fade Tags are used to fade in a block of text on screen instead of making it appear character by character. Similarly to colour tags, Fade tags have a start and end tag to denote which piece of text should appear instantly. The number is specified in centiseconds, which is 1/100th of a second.

Usage: `Hello, <i100>world!</i>`

![GIF showing fade tag](/assets/fadein_tag.gif)
