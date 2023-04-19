# Publishing Geode Mods

Once your awesome mod that adds sex to Geometry Dash is finished, it's time to publish it for all the world to see! Geode comes with an in-game "Download" section where users can download mods from, which gets its content from [the Geode Mods repo](https://github.com/geode-sdk/mods) (also known as the **mods index**)

## Getting Your Mod on the Repo

The mods repo uses an automated system, where the mods' built `.geode` packages are hosted on a repository of your own known as an **indexer**, and automatically merged into the main `mods` repository from there. To get started, build your mod, then run `geode project publish` in the project's directory (or `geode project publish -p <path/to/the/.geode>`, if it's built from a Github Action or the `.geode` package is otherwise located elsewhere than the project's `build` directory). Follow the instructions provided by CLI to create a fork of the indexer.

Once you have the fork created, you can publish mods by just running `geode project publish` in the mods' project directory after building it.

**If your mod is built with a Github Action** (for cross-platform support), you can provide an explicit path argument with `-p` to `geode project publish`.

`geode project publish` will tell you to open up a pull request on your indexer – do so to let us know you are ready to publish! Once your pull request has been made, one of the lead Geode developers will check that [it adheres to all guidelines](/source/guidelines.md), and approve the mod to the repo, or say what needs to be changed for it to be approved.

To release an update, just follow the same process as for publishing new mods.

## Guidelines

Getting a mod on the index requires a few things:

1. You haven't put a virus in your mod.

2. You're willing to put the mod up [for free](#what-about-paid-mods).

3. Your mod follows all other [required guidelines](/source/guidelines.md), and your mod passes the [Index review guidelines](/source/indexrules).

That's it. It's mostly formal stuff to make sure your mod is compatible with other mods and not dangerous. If your mod doesn't crash every five seconds, it's probably good to go.

If a mod on the index is later found to be breaking guidelines, for example it starts crashing or becomes incompatible with other mods (that it should be compatible with), we will let you know that you should update it. If the problems become severe and you don't do anything to fix the problem, we may remove your mod from the index.

## What about paid mods?

If your mod is paid, publish it yourself! Unfortunately, the Geode team doesn't want to handle processing user authentification and payments. You will have to set up your own system if you want to make your mod paid.

However, we still believe in paying creators for their work; if you want your mod on the index, but would still like to make some money out of it, you can [include a `support.md` file](/mods/publishing.md) in your mod that will show a big donate button in the mod's info page in-game. In there, you can include links to your PayPal, Patreon, OnlyFans, and whatnot :)
