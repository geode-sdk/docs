---
title: 1.2. Geode CLI
order: 3
---

# Geode CLI

Geode has its own CLI tool to aid in many tasks involved in making mods, such as packing assets, generating fonts, managing installed SDK versions, etc.. While it is technically possible to use Geode without the CLI, there is little reason not to install it as **it's required for nearly everything in practice**.

# Installation

* [Windows](#windows)
* [MacOS](#macos)
* [Linux](#linux)

## Windows

### Scoop

You can use [Scoop](https://scoop.sh/) to easily install the CLI by doing:
```bash
scoop bucket add extras
scoop install geode-sdk-cli
```
Later on, you can easily update the Geode CLI by doing:
```bash
scoop update geode-sdk-cli
```

### winget

If you prefer, you can also use [WinGet](https://learn.microsoft.com/en-us/windows/package-manager/winget/):
```bash
winget install GeodeSDK.GeodeCLI
```
To update the CLI, run:
```bash
winget upgrade GeodeSDK.GeodeCLI
```

---

**(Not Recommended!)** Otherwise, you can manually install the Geode CLI by:
1. Download the latest Windows release over on [GitHub](https://github.com/geode-sdk/cli/releases/latest)
1. Extract `geode.exe` into some folder on your computer
1. Select the executable in File Explorer, Shift + Right-Click it and select `Copy as Path`
1. Search `Edit the system environment variables` on Windows search. Alternatively, you can open up Control Panel and search for it, then select `Edit the system environment variables` or **to skip straight to step 6 select `Edit environment variables for your account`**.
1. Click `Environment Variables...`
1. In the top `User variables` section, select the `Path` variable and click `Edit`
1. Now click `New` and paste the path of the CLI executable you copied at Step 1. **Remove the `\geode.exe` from the end;** the path has to point to the _directory_ with Geode CLI, not the CLI itself.
1. Click OK to close the environment variable windows.

---

After installing the CLI, you should now be able to run `geode --version` in your command line and see a version number! (If this doesn't work, try restarting your terminal and/or computer.)

It is recommended that you [set up a profile afterwards](#profile-setup).

## MacOS

You can easily install the CLI via [Brew](https://brew.sh)
```bash
brew install geode-sdk/geode/geode-cli
```

It is recommended that you [set up a profile afterwards](#profile-setup).

## Linux

We provide prebuilt Linux binaries in [the CLI releases page](https://github.com/geode-sdk/cli/releases/latest). Since Linux distros differ between each other, you need to figure out yourself how to add this binary to your global path so CMake can find it. As long as `geode --version` works anywhere, everything should be fine.

Once you figure that out, it is recommended that you [set up a profile afterwards](#profile-setup).

# Profile Setup

A profile is just an instance of Geometry Dash. The CLI allows keeping multiple separate installations of Geometry Dash at once, though most users will just have a single installation of GD with Geode on it. If you do have GDPSes with Geode on them installed, you can run `geode profile add` to add them to the list of known profiles. You need to have at least one profile set up so your mods can be automatically installed post build.

To setup a new profile, simply run the `geode config setup` command on your terminal.
