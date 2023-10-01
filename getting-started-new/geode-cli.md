---
title: 1.2. Geode CLI
order: 3
---

# Geode CLI
The Geode SDK has its own command line utility program to aid in many tasks involved in mod, such as asset packing, font generation, managing the SDK, etc.

# Installation

* [Windows](#windows)
* [MacOS](#macos)

## Windows
If you have [scoop](https://scoop.sh/) you can easily install the cli by doing:
```bash
scoop install https://geode-sdk.org/geode-sdk-cli.json
```

---

Otherwise, you can manually install it by:
1. Download the latest windows release over at [GitHub](https://github.com/geode-sdk/cli/releases/latest)
1. Extract the `geode.exe` into some folder on your computer
1. Select the CLI executable in File Explorer, Shift + Right-Click it and select `Copy as Path`
1. Search `Edit the system environment variables` on Windows search. Alternatively, you can open up Control Panel and search for it, then select `Edit the system environment variables` or **to skip straight to step 6 select `Edit environment variables for your account`**.
1. Click `Environment Variables...`
1. In the top `User variables` section, select the `Path` variable and click `Edit`
1. Now click `New` and paste the path of the CLI executable you copied at Step 1. **Remove the `\geode.exe` from the end;** the path has to point to the _directory_ with Geode CLI, not the CLI itself.
1. Click OK to close the environment variable windows.

---

After either way of installing it, you should now be able to run `geode --version` in your cmd and see a version number!

It is recommended that you [set up a profile now](#profile-setup).

## MacOS

<!-- TODO: -->
i dont know either sorry mac ppl

It is recommended that you [set up a profile now](#profile-setup).

# Profile Setup

A profile is just an instance of Geometry Dash. It's a good idea to set up one up for CLI so your mods can be automatically installed post build.

To setup a new profile, simply run the `geode config setup` command on your terminal.