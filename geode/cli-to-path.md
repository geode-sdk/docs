# Adding CLI to PATH

In order for the CLI to be accessible from anywhere on your computer, it needs to be added to your `PATH` environment variable. If you know what that means, you know how to do it; otherwise, follow these steps:

## On Windows

1. Select the CLI executable in File Explorer, Shift + Right-Click it and select `Copy as Path`

2. Search `Edit the system environment variables` on Windows search. Alternatively, you can open up Control Panel and search for it, then select `Edit the system environment variables` or **to skip straight to step 3 select `Edit environment variables for your account`**.

3. Click `Environment Variables...`

4. In the top `User variables` section, select the `Path` variable and click `Edit`

5. Now click `New` and paste the path of the CLI executable you copied at Step 1. **Remove the `\geode.exe` from the end;** the path has to point to the _directory_ with Geode CLI, not the CLI itself.

6. Click OK to close the environment variable windows.

### Making sure it works

1. Open up Windows search and open `cmd` or `powershell`

2. Type `geode` and hit Enter. If CLI was installed correctly, you should see the CLI help displaying its version and commands.

If the command didn't work, try restarting the terminal you have open, or restarting your computer.

## On Mac

1. While holding down Control on your keyboard, Select the CLI executable in the Finder app. 

2. While holding down Option/Alt on your keyboard, click `Copy "geode" as Pathname`

3. Now press Command + Spacebar to open the MacOS search, type `Terminal`, and press Enter. This should bring up the Terminal application in a new window.

4. In the Terminal window, write `export PATH="my/path/to/folder:${PATH}"` replacing `my/path/to/folder` with the path of the CLI executable you copied at Step 2. **Remove the `\geode` from the end;** the path has to point to the _directory_ with Geode CLI, not the CLI itself.

5. Press Enter on your keyboard to execute the command.

### Making sure it works

1. On that same Terminal window, type `geode` and hit Enter. If CLI was installed correctly, you should see the CLI help displaying its version and commands.

If the command didn't work, try restarting the terminal you have open, or restarting your computer.