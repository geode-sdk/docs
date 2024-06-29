---
title: 2. Setting up the SDK
order: 4
---

# Setting up the SDK
To install the SDK we will be using the CLI installed on the previous step.

To download the sdk, you can simply run the command:
```bash
geode sdk install
```
This *should* set the `GEODE_SDK` enviroment variable, which can you test after reopening your terminal:
```bash
# on windows
echo %GEODE_SDK%

# elsewhere
echo $GEODE_SDK
```

If that command prints out the path you installed the SDK to, then it has worked correctly.

To develop mods, you should also download the prebuilt binaries for Geode, which you can do by running this command:
```bash
geode sdk install-binaries
```

## Cache

It is **highly** recommended to set the [CPM_SOURCE_CACHE](https://github.com/cpm-cmake/CPM.cmake?tab=readme-ov-file#cpm_source_cache) environment variable. This will prevent CMake from cloning repositories you've already cloned, and allow you to build mods offline (given you have built them online at least once).

To do this, create a directory somewhere permanent, and set the environment variable `CPM_SOURCE_CACHE` to the full path to that folder.

## Updating
You will need to manually update your local SDK every once in a while, which you do by running this command:
```bash
geode sdk update
```

Every time you update the SDK, you should update its prebuilt binaries too.
```bash
geode sdk install-binaries
```

---

You can also switch to the **nightly** version, which uses the latest commit.
```bash
geode sdk update nightly
```

Or to go back to stable:
```bash
geode sdk update stable
```