# Tips for Getting Your Mod Approved

These are an unsorted collection of some tips for making sure your mod adheres to the [index guidelines](/mods/guidelines.md).

## Common pitfalls

While none of the things in this section are absolute rules, they are red flags that index staff look for while reviewing a mod. These pitfalls describe common issues that may cause crashes or incompatibility with other mods/the game.

* Avoid generating exceptions.

Due to compatibility issues, functions that generate C++ exceptions should be avoided, _even if you catch the exception_ (since try/catch blocks **don't work** on all platforms). Either verify that the exception will not be generated beforehand, or use an alternative function that does not throw one. One such function is `std::stoi` and `std::stof`, which should be replaced with Geode's [`numFromString`](/functions/geode/utils/numFromString/).

Functions from the `<filesystem>` header should be handled carefully. Most filesystem functions have `std::error_code` overloads, which do not throw exceptions on failure. **You should use error_code overloads**! Calling `string()` on a `std::filesystem::path` must also be avoided - this may crash on certain setups. Instead, use [`pathToString`](/functions/geode/utils/string/pathToString/).

Calling `unwrap()` on a [Result](/classes/geode/Result/) without first checking if it is `ok()` will lead to an instant rejection. You can never be certain that your web calls will always succeed!

* Avoid usage of `dynamic_cast`.

Due to how dynamic casting is implemented, usage of this function will usually fail for classes from other mods and the game. Instead, use [`typeinfo_cast`](/functions/geode/cast/typeinfo_cast/).

* Avoid blocking the main thread.

While it is unavoidable in some cases (or will cause additional instability), offloading expensive functions to a separate thread avoids the dreaded "application not responding" and greatly improves the user experience. [Tasks](/tutorials/tasks) were designed for this use case and can be used to wrap any callback-based async api. Mods blocking the main thread for web requests will be instantly rejected - please use Geode's [web utils](/tutorials/fetch).

* Avoid reinventing the wheel.

Hooking functions, especially heavily called ones, to accomplish tasks that can be done by other means may get your mod rejected. One example would be hooking `CCScheduler::update` to run code on each frame (use [`scheduleSelector`](/classes/cocos2d/CCScheduler#scheduleSelector) instead) or to keep a node across scenes (use [`SceneManager::keepAcrossScenes`](/classes/geode/SceneManager#keepAcrossScenes)). Besides creating performance issues, this may also reduce compatibility with other mods.

While this isn't typically a rejection reason, new developers tend to find themselves "reinventing the wheel" and using platform-specific functions to accomplish tasks that are already covered by Geode utils. For example, [restarting the game](/functions/geode/utils/game/restart/), [opening a file picker](/functions/geode/utils/file/pick) and [splitting a string](/functions/geode/utils/string/split) are all built into the Geode utils. These functions have the most compatibility with other mods and are tested across all supported platforms. A basic list of them can be found [here](/tutorials/utils).

Developers should also be aware of Geode's [directory functions](https://github.com/geode-sdk/geode/blob/main/loader/include/Geode/loader/Dirs.hpp), which can be used for storing mod files and determining the location of game data. Unless you have good reason, mod files should be stored within the mod's save directory, which can be determined with [`Mod::getSaveDir`](/classes/geode/Mod#getSaveDir) (using the config directory through `getConfigDir` is also fine).

* Avoid submitting **locally built mods**.

You should be using the cross-platform [build-geode-mod](https://github.com/geode-sdk/build-geode-mod/) action that comes with the default mod template. This action helpfully builds your mod for all platforms, which makes it very easy for developers to support them. This will be the `.github/workflows/multi-platform.yml` file in your repo.

If this doesn't exist, you likely said no to adding it when making the geode project (though this option has since been removed, and it will always be added). Add the [`multi-platform.yml`](https://github.com/geode-sdk/build-geode-mod/blob/main/examples/multi-platform.yml) file from the examples folder into `.github/workflows/multi-platform.yml`, then check the Actions tab in GitHub and take the Build Output from the most recent commit, take the .geode file out of it and place that in the release.

Those who are uploading their code on platforms outside of GitHub should also be using the CI instances offered by those platforms for building their mod. This makes it easier for us to verify that a mod has not been tampered with.
