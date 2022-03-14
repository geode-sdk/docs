---
layout: default
title: How Geode works
nav_order: 1
description: "Fundamentals of Geode"
---

# How Geode works

Geode as a framework is fundamentally built as a low-level framework with user-friendly programming patterns. This means that the user is still permitted to do low-level operations such as hooking arbitary addresses and patching arbitary bytes, while Geode also provides an easy interface for doing common operations. This approach grants the user the near-endless flexibility of a low-level framework and the simplicity of a higher-level abstracted interface.

Geode mods are not like normal standalone GD mods. The Geode loader, while technically capable of loading arbitary binaries, does not have any direct API for doing so as loading arbitary binaries is both a security concern and may cause problems. Instead, the files that Geode loads are known as `.geode` files, which are ZIP files that contain all of the mod's resources and code.

Every `.geode` file has at least 2 things: a `mod.json` file and a platform binary. The `mod.json` file contains metadata and other info about a mod, such as its ID, name, resources, settings, and others. The platform binary is the actual mod code; it is what on Windows is known as a DLL, on Mac / iOS a dylib and on Android an SO.

As `.geode` files are just ZIP files, one may be created by simply using any ZIP-creation utility such as the "Send to Compressed Folder" option on Windows File Explorer, and renaming the file to have the `.geode` extension. However, for developing mods where one may recompile the platform binary dozens if not hundreds of times, this may prove to be quite cumbersome. For that reason, Geode also comes with a command-line utlitity known as the **Geode CLI** which has the `pkg` command for creating `.geode` files based on a `mod.json` file. If you're using the default Geode build tools, you will likely never need to run the `pkg` command yourself however, as the SDK's CMake files automatically call `pkg` on a built binary as long as you have the Geode CLI located somewhere within the `PATH` environment variable.


