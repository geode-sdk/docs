---
title: iOS Development Setup
---

# iOS Development Setup

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
geode sdk install-binaries --platform ios
```

SDK Version 4.4.0 or higher is required, so make sure to update your SDK first.

Now you can build your mod for iOS via:
```bash
geode build -p ios
```
Or if you want to build manually:
```bash
cmake -B build -DCMAKE_SYSTEM_NAME=iOS -DGEODE_TARGET_PLATFORM=iOS -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build build
```

## Github Actions
To build your mod for iOS using `geode-sdk/build-geode-mod`, you have to add iOS to the build matrix, like so:
```yml
- name: iOS
  os: macos-latest
  target: iOS
```

## Web Server

If your iOS device is not jailbroken, it is generally recommended to follow this section to enable the web server.

> :warning: The web server for the TrollStore version of the launcher will not work properly. It is recommended to use `scp` to upload your mods instead.

The web server is a feature bundled into the **Geode Launcher** that allows you to easily upload mods without needing to use iTunes. This eliminates the need to manually drag your geode mod into the mods directory, making the mod testing process more simpler.

To activate the web server:
1. Open the launcher and navigate to settings
2. Scroll down until you reach the **About** section
3. Hold on the **iOS Launcher** text for at least 3 seconds until a popup appears prompting you to enable **Developer Mode**
4. Tap **Yes**, then scroll all the way down to find **Web Server**
5. Enable the setting and restart the launcher
![Image showing the steps above but a visualization of it](/assets/Misc_iOS_webserver-steps.png)

### Using the Web Server

After restarting the launcher, navigate to `http://[Your Device IP]:8080` in your browser on your computer, replacing `[Your Device IP]` with the local IP address of your iOS device.

Now you should see a web interface for the launcher in your browser!
![Image showcasing the web interface on browser](/assets/Misc_iOS_webserver-ui1.png)

Through this web interface, you can:
- Upload your mods
- Launch the game
- View in-game logs if the game has been launched

As an example, this is what the web interface would look like when the game is launched:
![Image showcasing the web interface on browser but with game logs](/assets/Misc_iOS_webserver-ui2.png)

### Other Upload Method

If you prefer not to upload your mods through your browser, you can use a script to upload your mod to your device, and launch the game immediately after you compile and upload your mod:
```bash
# Build commands here...

# Replace the Example ID with your Mod ID, or the path to your compiled geode file. This is assuming the CWD is the project.
GEODE_MOD="./build-ios/my.example-mod.geode"

# Replace the device URL with your device's local IP address
DEVICE_URL="http://192.168.0.25:8080"
curl -X POST -F "file=@${GEODE_MOD}" "${DEVICE_URL}/upload"

# Including this curl request is optional.
curl -X POST "${DEVICE_URL}/launch"
```
