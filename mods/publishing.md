# Publishing Geode Mods

Once your awesome mod is finished, it's time to publish it for all the world to see! Geode comes with an in-game "Download" section where users can download mods from, which gets its content from [the Geode Index](https://api.geode-sdk.org/v1/mods).

## Getting Your Mod on the Repo

> :warning: All mods submitted on the index **must provide the source code**! If your mod is open source, just include a link to a Github repository or equivalent. For closed source mods, see [the dedicated section](#what-about-closed-source-mods)

Submitting a mod to the official mod index is as follows:

1. Make sure you have the [**latest** CLI](/getting-started/geode-cli) set up.
2. Build and release your mod somewhere - we highly recommend using GitHub releases, as this provides a straight-forward way to deal with versioning.
   - Do **NOT** replace existing uploaded versions! This will change the hash and thus users will be unable to download the old version.
4. Login to the index using the **CLI**: `geode index login`. This will prompt you to login using your GitHub account.
5. Run `geode index mods create`
6. Provide a **direct download link** to the .geode file (for example `https://github.com/HJfod/BetterEdit/releases/download/v6.3.3/hjfod.betteredit.geode`)
7. An **index admin** will have to validate that your mod meets the [index guidelines](/mods/guidelines) and approve your mod.

## Releasing updates

To release an update, use `geode index mods update`. You will have to be already logged in with the index to use this command.

If you are using GitHub releases (or any other system), **do not update an existing release** - create a new one instead. Updating an existing release **will break that version of the mod**, as the Geode package is checksummed.

Make sure to **increase your mod version** when updating it! You should be following [Semantic Versioning](https://semver.org), especially if you're developing a mod with a **public API**.

If you are a **verified** developer, then the update will automatically be accepted onto the index, without the need of approval from an **index admin**.

## What are **verified** developers?

Being a verified developer has the sole benefit of not requiring your mod updates to be checked by an **index admin** (and a cool role on the Discord server). **Newly created mods** will still have to go through the verification process. Of course, this means that we do not give the verified status easily. You can become a verified developer by having good quality mods (or mod, you don't have to upload tens of them), proving that you are trustworthy, and being an overall helpful figure in the modding community.

Once you have your verified status, try and uphold the same quality standards you were keeping before getting the role. Having no update checks comes with the temptation of releasing updates **faster** and "figuring it out later if it breaks", which is not a mindset you should be having. Consistently having incidents where you release a broken update **will** lead to your verified status being removed. It also goes without saying that if you release mods / updates that break the **ban rules**, there will be consequences.

## What about closed source mods?

Even if your mod is **closed source**, you still need to submit the source code for verification. Do so by asking [someone who can approve new mods](#who-can-approve-mods) on the index repository and send them the source code privately, for example by adding them temporarily as a contributor to your private repository.

**Geode developers will never leak or steal your source code!** If you find that the person who verified your mod has breached this vital level of trust, **do let the other Geode lead developers know immediately**.

## What about paid mods?

Paid mods may be published on the index, as long as they follow the [guidelines for paid mods](/mods/guidelines#paid-mods). Do note that the rules for closed source mods still apply to paid mods; paid mods must submit their source code for verification, including any extra closed-source components that are downloaded at runtime.

## Who can approve mods?

**New mods** and **updates** for mods can be verified by **lead developers** and **index admins**. Additionally, anyone with the `verified` priviledge can automatically submit new mods / update their own mods without needing to wait for a staff member to verify it.
