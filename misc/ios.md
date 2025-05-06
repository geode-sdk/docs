---
title: iOS Development Setup
---

# iOS Development Setup

Something something preamble

You will have to edit your `CMakeLists.txt` on your exiting mods. Newer created mods will already have this change
```cmake
# At the beginning of your CMakeLists.txt, change this:
#  set(CMAKE_OSX_ARCHITECTURES "arm64;x86_64")
# into:
if ("${CMAKE_SYSTEM_NAME}" STREQUAL "iOS" OR IOS)
    set(CMAKE_OSX_ARCHITECTURES "arm64")
else()
    set(CMAKE_OSX_ARCHITECTURES "arm64;x86_64")
endif()
```

## Build

To build mods for iOS, you must have Mac OS with the iPhone SDK installed from Xcode.

You must also get the Geode binaries for iOS, you can do this using the CLI:
```bash
geode sdk update
geode sdk install-binaries --platform ios
```

Now you can build your mod for iOS via:
```bash
geode build -p ios
```
Or if you want to build manually:
```bash
cmake -B build -DCMAKE_SYSTEM_NAME=iOS -DGEODE_TARGET_PLATFORM=iOS -DCMAKE_BUILD_TYPE=RelWithDebInfo -G Ninja
cmake --build build
```

## Github Actions
To build your mod for iOS using `geode-sdk/build-geode-mod`, you have to add iOS to the build matrix, like so:
```yml
- name: iOS
  os: macos-latest
  target: iOS
```
