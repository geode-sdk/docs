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


## Github Actions
To build your mod for iOS using `geode-sdk/build-geode-mod`, you have to add iOS to the build matrix, like so:
```yml
- name: iOS
  os: macos-latest
  target: iOS
```

As of writing this, iOS support is only available on nightly Geode, so make sure to add this to the options of the `build-geode-mod` step
```yml
sdk: nightly
```