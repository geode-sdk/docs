---
layout: default
title: Installing Geode CLI
parent: Getting Started
nav_order: 1
description: "How to set up Geode CLI"
---

# Installing Geode CLI

Geode CLI is the command-line interface for working with Geode, intended mainly for developers. It comes with many important tools such as packaging mods into `.geode` files, creating spritesheets, creating bitmap fonts, etc.. The CLI, if installed, will be automatically invoked by Geode CMake files, so you likely won't need to use it a lot manually; however, for Geode mod development, it is very much recommended to have it installed as you will have to manually create resources and `.geode` files otherwise.

## Downloading CLI

### Latest Release

You can find the latest release of CLI on the [Release page on its repo](https://github.com/geode-sdk/cli). Do note that this release may be a little outdated; please [let us know](/docs/contributing) if it seems to be missing features, and we'll be sure to create a new release.

### Building from source

You can build the CLI yourself using [Rust](https://doc.rust-lang.org/cargo/getting-started/installation.html). Clone the git repo using `git clone https://github.com/geode-sdk/cli` and then run `cargo build` to build it. Do note that before you can run CLI on Windows, you need to manually move `libgeode.dll` from the `target/debug/deps` directory to `target/debug` (we don't know why Rust doesn't move it for us). You can do this with the command `copy target/debug/deps/libgeode.dll target/debug`.

## Adding CLI to PATH

> :warning: Note: this section is for Windows.

In order for the CLI to be accessible from anywhere on your computer, it needs to be added to your `PATH` environment variable. If you know what that means, you know how to do it; otherwise, follow these steps:

1. Select the CLI executable in File Explorer, Shift + Right-Click it and select `Copy as Path`

2. Search `Edit the system environment variables` on Windows search. Alternatively, you can open up Control Panel and search for it, then select `Edit the system environment variables` or **to skip straight to step 3 select `Edit environment variables for your account`**.

3. Click `Environment Variables...`

4. In the top `User variables` section, select the `Path` variable and click `Edit`

5. Now click `New` and paste the path of the CLI executable you copied at Step 1. Remove the `\geode.exe` from the end.

6. Click OK to close the environment variable windows.

## Making sure it works

1. Open up Windows search and open `cmd` or `powershell`

2. Type `geode` and hit Enter. If CLI was installed correctly, you should see the CLI help displaying its version and commands.

