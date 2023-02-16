---
title: FAQ
description: Answers to common questions about Geode
icon: info
---

# Frequently Asked Questions

## How do I install Geode?

You can install Geode from [the Official Homepage](https://geode-sdk.org). If you're looking to develop your own mods using Geode, see [Installation](/installation.md).

## I already have Mega Hack / QuickLdr / GDHM, how do I migrate?

The Geode installer should detect and remove old mod loaders automatically. You can then reinstall your mods through the in-game Download page, if they are available.

## How do I use Mega Hack with Geode?

Absolute has said the next major release of Mega Hack (v8) will be available on Geode. The current version [won't work alongside Geode](#how-do-i-load-dll-mods), so you will have to wait for v8 to be released.

## How do I load .DLL mods?

Geode does not support loading old .DLL mods. Geode will most likely not work if installed alongside another mod loader either. Check if the mods you want to install are available as Geode mods.

## Why doesn't Geode support .DLL mods?

Short answer: Because if it did support it, people would complain about everything crashing.

Geode introduces, among a lot of other features, a brand new hooking framework that is incompatible with older mods, which often use MinHook. As such, loading older mods while Geode is loaded will likely cause issues. On top of the hooking framework, Geode also shuffles a lot of UIs around internally, which is fine for Geode mods but will completely break older mods.

## When will [insert mod name] get ported?

Depends on the developer. If they're just porting the existing codebase with no changes, porting might only take a day. However, most mods such as BetterEdit and TextureLdr are getting extra features added while being ported, and as such will take longer. Follow the developer's social medias to see if they post progress updates.

Once a mod has been ported, you can find it on the in-game Download page.

## Help! The game crashed!

Please send the latest crashlog from `geode/crashlogs` to us [on Github issues](https://github.com/geode-sdk/geode/issues/new/choose).
