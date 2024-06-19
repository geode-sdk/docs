# Publishing Geode Mods

Once your awesome mod is finished, it's time to publish it for all the world to see! Geode comes with an in-game "Download" section where users can download mods from, which gets its content from [the Geode Index](https://api.geode-sdk.org/v1/mods).

## Getting Your Mod on the Repo

> :warning: All mods submitted on the index **must provide the source code**! If your mod is open source, just include a link to a Github repository or equivalent. For closed source mods, see [the dedicated section](#what-about-closed-source-mods)

Submitting a mod to the official mod index is as follows:

1. Download the latest release of the [CLI](https://github.com/geode-sdk/cli/releases/latest). You can also use [Scoop](https://scoop.sh/), or a [brew cask on macOS](https://github.com/geode-sdk/cli/releases/latest). Also make sure the CLI is added to your **path**
2. Build and release your mod somewhere - we highly recommend using GitHub releases, as this provides a straight-forward way to deal with versioning.
3. Login to the index using the **CLI**: `geode index login`. This will prompt you to login using your GitHub account.
4. `geode index mods update`
5. Provide a **direct download link** to the .geode file (for example `https://github.com/HJfod/BetterEdit/releases/download/v6.3.3/hjfod.betteredit.geode`)
6. If you are **verified** on the index, then the mod will be available to download immediately. If you are not verified, then an **index admin** will have to validate that your mod meets the [index guidelines](/mods/guidelines) and approve your mod.

## Releasing updates

To release an update, use `geode index mods update`. You will have to be already logged in with the index to use this command.

If you are using GitHub releases (or any other system), **do not update an existing release** - create a new one instead. Updating an existing release **will break that version of the mod**, as the Geode package is checksummed.

Make sure to **increase your mod version** when updating it! You should be following [Semantic Versioning](https://semver.org), especially if you're developing a mod with a public API.

## What about closed source mods?

Even if your mod is **closed source**, you still need to submit the source code for verification. Do so by asking [someone who can approve new mods](#who-can-approve-mods) on the index repository and send them the source code privately, for example by adding them temporarily as a contributor to your private repository.

**Geode developers will never leak or steal your source code!** If you find that the person who verified your mod has breached this vital level of trust, **do let the other Geode lead developers know immediately**.

## What about paid mods?

If your mod is fully paid, you will have to **publish it yourself on a separate platform**, as the Geode team doesn't want to handle processing user authentification and payments. However, there are many options for still getting the mod on the index:

 - If your mod has a **free / lite version** that is usable on its own (like [BetterEdit](https://github.com/HJfod/BetterEdit) does), then publishing that is on the index is perfectly okay! In the same vain, if an otherwise free mod **contains paid features**, that is allowed - you just have to **make it clear that the mod contains paid content** in the mod description. In the future you will also be required to add a `Contains Paid Content` tag for free mods that have paid features.

 - If your mod has no free version, you may also publish an **installer mod** for your paid mod on the index that just asks the user to authenticate / purchase the mod and installs the paid version. However, you will have to make it **unambiguously and obviously clear that the mod is a link to a paid mod before the user installs it**, for example by naming the mod "Trial Version" or equivalent. In the future you will also be required to add a `Paid` tag for mods that act as just wrapper for paid mods.

 - All mods can also include a [`support.md` file](/mods/md-files#supportmd) in your mod that will show a big donate button in the mod's info page in-game. In there, you can include links to your PayPal, Patreon, OnlyFans, and whatnot :)

## Who can approve mods?

**New mods** and **updates** for mods can be verified by **lead developers** and **index admins**. Additionally, anyone with the `verified` priviledge can automatically submit new mods / update their own mods without needing to wait for a staff member to verify it.
