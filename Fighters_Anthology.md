# Jane's Fighters Anthology with ControllerBuddy on Linux

## 📖 Description

This guide describes the installation of [Jane's Fighters Anthology](https://en.wikipedia.org/wiki/Fighters_Anthology) for use with [ControllerBuddy](https://controllerbuddy.org/) on Linux via the [Proton](https://github.com/ValveSoftware/Proton) compatibility layer.

What you get with this setup:
- An installation of Jane's Fighters Anthology that is nicely integrated into your Steam library.
- ControllerBuddy will start automatically when you start the game, load the correct profile, and exit after you exit the game.

## 🧩 Prerequisites

- [Steam](https://steampowered.com/) (distribution package)
- [protontricks Flatpak](https://flathub.org/en/apps/com.github.Matoking.protontricks)
- [ControllerBuddy Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak)
- `FA_1_00F.iso` (Jane's Fighters Anthology installation disc ISO image)

## 🪜 Steps

1. Mount the `FA_1_00F.iso` disc image to a temporary directory:
```sh
export CDROM_DIR=$(mktemp -d cdrom-XXXX) &&
sudo mount -o loop FA_1_00F.iso "$CDROM_DIR" &&
echo "setup.exe located at: $CDROM_DIR/setup.exe"
```

> [!IMPORTANT]
> All subsequent commands must be executed within the same shell session to retain the `CDROM_DIR` environment variable.

2. Add `setup.exe` as a Non-Steam game
3. Rename the **setup.exe** Steam shortcut to **Jane's Fighters Anthology**.
4. Select **Proton 9.0-4** as the compatibility tool.
5. Launch the **Jane's Fighters Anthology** Steam shortcut and install Fighters Anthology.  
    During the setup, select "Full Install - Digital Music".
6. Obtain the `APP_ID` of the Proton prefix:
    ```sh
    export APP_ID=$(flatpak run com.github.Matoking.protontricks -l \
        | grep "^Non-Steam shortcut: Jane's Fighters Anthology ([0-9]\+)$" \
        | sed -E 's/.*\(([0-9]+)\).*/\1/' \
        | head -n1)
    echo "APP ID: $APP_ID"
    ```

> [!IMPORTANT]
> All subsequent commands must be executed within the same shell session to retain the `APP_ID` environment variable.

7. Download the `fae102.exe` with your browser:
    ```sh
    xdg-open https://jkpeterson.net/fa/misc/fae102.exe &
    ```

8. Install the `fae102.exe` patch using Protontricks:
    ```sh
    cd $(xdg-user-dir DOWNLOAD) &&
    flatpak run com.github.Matoking.protontricks -c 'wine fae102.exe' "$APP_ID"
    ```

> [!IMPORTANT]
> During the patch installation, `C:\JANES\Fighters Anthology\FA.EXE` must be selected via **My Computer**.

9. Download the `FAno-cd.zip` no-disc patch with your browser:
    ```sh
    xdg-open https://jkpeterson.net/fa/misc/FAno-cd.zip &
    ```

10. Extract the `FAno-cd.zip` no-disc patch to your Fighters Anthology installation directory:
    ```sh
    cd $(xdg-user-dir DOWNLOAD) &&
    unzip -o FAno-cd.zip -x README.txt -d "/home/$USER/.local/share/Steam/steamapps/compatdata/$APP_ID/pfx/drive_c/JANES/Fighters Anthology"
    ```

11. Replace the fake `fa_7.lib` of the no-disc patch with `fa_7.lib` from the disc image:
    ```sh
    cp "$CDROM_DIR/fa_7.lib" "/home/$USER/.local/share/Steam/steamapps/compatdata/$APP_ID/pfx/drive_c/JANES/Fighters Anthology/"
    ```

> [!IMPORTANT]
> Do not replace the fake `fa_4c.lib` with the one from the disc, as it will cause a crash to desktop during startup.

12. Unmount the `FA_1_00F.iso` file and delete the temporary directory:
    ```sh
    sudo umount "$CDROM_DIR" &&
    rm -rf "$CDROM_DIR"
    ```

12. Hide all game controllers from the Proton prefix, except for ControllerBuddy's UINPUT joystick device:
    ```sh
    reg_file=$(mktemp joysticks-XXXX.reg) &&
    python3 - <<'EOF' "$reg_file" &&
    import ctypes
    import ctypes.util
    import sys

    sdl2_path = ctypes.util.find_library("SDL2")
    if not sdl2_path:
        raise RuntimeError("Could not find SDL2 library")

    sdl = ctypes.CDLL(sdl2_path)

    Uint16 = ctypes.c_uint16
    Uint32 = ctypes.c_uint32
    class SDL_JoystickGUID(ctypes.Structure):
        _fields_ = [("data", ctypes.c_uint8 * 16)]

    sdl.SDL_Init.argtypes = [ctypes.c_uint32]
    sdl.SDL_Init.restype = ctypes.c_int

    sdl.SDL_Quit.argtypes = []
    sdl.SDL_Quit.restype = None

    sdl.SDL_NumJoysticks.argtypes = []
    sdl.SDL_NumJoysticks.restype = ctypes.c_int

    sdl.SDL_JoystickNameForIndex.argtypes = [ctypes.c_int]
    sdl.SDL_JoystickNameForIndex.restype = ctypes.c_char_p

    sdl.SDL_JoystickGetDeviceGUID.argtypes = [ctypes.c_int]
    sdl.SDL_JoystickGetDeviceGUID.restype = SDL_JoystickGUID

    sdl.SDL_GetJoystickGUIDInfo.argtypes = [SDL_JoystickGUID, ctypes.POINTER(Uint16), ctypes.POINTER(Uint16), ctypes.POINTER(Uint16), ctypes.POINTER(Uint16)]
    sdl.SDL_GetJoystickGUIDInfo.restype = None

    SDL_INIT_JOYSTICK = 0x00002000

    if sdl.SDL_Init(SDL_INIT_JOYSTICK) != 0:
        print("SDL_Init failed:", sdl.SDL_GetError().decode("utf-8"))
        sys.exit(1)

    # see: https://github.com/wine-mirror/wine/blob/master/dlls/hidclass.sys/device.c
    device_strings = {
        (0x045E, 0x028E): "Controller (XBOX 360 For Windows)",
        (0x045E, 0x028F): "Controller (XBOX 360 For Windows)",
        (0x045E, 0x02D1): "Controller (Xbox One For Windows)",
        (0x045E, 0x02DD): "Controller (Xbox One For Windows)",
        (0x045E, 0x02E3): "Controller (Xbox One For Windows)",
        (0x045E, 0x02EA): "Controller (Xbox One For Windows)",
        (0x045E, 0x02FD): "Controller (Xbox One For Windows)",
        (0x045E, 0x0719): "Controller (XBOX 360 For Windows)",
        (0x045E, 0x0B00): "Controller (Xbox One For Windows)",
        (0x045E, 0x0B05): "Controller (Xbox One For Windows)",
        (0x045E, 0x0B12): "Controller (Xbox One For Windows)",
        (0x045E, 0x0B13): "Controller (Xbox One For Windows)",
        (0x054C, 0x05C4): "Wireless Controller",
        (0x054C, 0x09CC): "Wireless Controller",
        (0x054C, 0x0BA0): "Wireless Controller",
        (0x054C, 0x0CE6): "Wireless Controller",
        (0x054C, 0x0DF2): "Wireless Controller",
    }

    count = sdl.SDL_NumJoysticks()
    joysticks = []

    for i in range(count):
        name_ptr = sdl.SDL_JoystickNameForIndex(i)
        if not name_ptr:
            continue
        name = name_ptr.decode("utf-8")

        if name == "ControllerBuddy Joystick":
            continue

        guid = sdl.SDL_JoystickGetDeviceGUID(i)
        vendor = Uint16()
        product = Uint16()
        version = Uint16()
        crc16 = Uint16()
        sdl.SDL_GetJoystickGUIDInfo(guid, ctypes.byref(vendor), ctypes.byref(product), ctypes.byref(version), ctypes.byref(crc16))

        key = (vendor.value, product.value)
        if key in device_strings:
            name = device_strings[key]

        joysticks.append(name)

    sdl.SDL_Quit()

    if not joysticks:
        print("Error: No joysticks detected on this system.", file=sys.stderr)
        sys.exit(1)

    def escape_reg_string(s: str) -> str:
        return s.replace('"', '""')

    print("Found the following joysticks:")
    for name in joysticks:
        print(f" {name}")

    if len(sys.argv) > 1:
        path = sys.argv[1]
        with open(path, "w", encoding="utf-16") as f:
            f.write("\ufeffWindows Registry Editor Version 5.00\n\n")
            f.write("[HKEY_CURRENT_USER\\Software\\Wine\\DirectInput\\Joysticks]\n")
            for name in joysticks:
                safe_name = escape_reg_string(name)
                f.write(f"\"{safe_name}\"=\"disabled\"\n")
        print(f"Wrote registry file: {path}")
    EOF
    flatpak run com.github.Matoking.protontricks -c "wine reg import '$reg_file'" "$APP_ID"
    rm -f "$reg_file"
    ```

> [!IMPORTANT]
> In the following step the placeholders denoted by `<...>` must be replaced accordingly to the following table:
> | Placeholder | Description                                     |
> |-------------|-------------------------------------------------|
> | `<USER>`    | Your username                                   |
> | `<APP_ID>`  | The Proton prefix **APP ID** obtained in step 6 |

13. Update the **Jane's Fighters Anthology** Steam shortcut as follows:

    **TARGET**:
    ```
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/JANES/Fighters Anthology/FA.EXE"
    ```

    **START IN**:
    ```
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/JANES/Fighters Anthology"
    ```

    **LAUNCH OPTIONS**:
    ```sh
    ${STEAM_RUNTIME}/scripts/switch-runtime.sh --runtime='' -- flatpak run de.bwravencl.ControllerBuddy -autostart local -profile /app/share/ControllerBuddy-Profiles/Fighters_Anthology.json -tray & timeout 10 bash -c 'until grep -q "ControllerBuddy Joystick" /proc/bus/input/devices ; do sleep 1 ; done' && %command% || { [ $? -eq 124 ] && zenity --error --text="Launch aborted because ControllerBuddy wasn't ready within 10 seconds.\n\nCheck if your controller is connected." --width 500 ; } ; killall -q ControllerBuddy
    ```

## 🎮 Steam Deck Specifics

### Launching from Gaming Mode

To allow launching Jane's Fighters Anthology with ControllerBuddy from the Steam Deck's Gaming Mode, a custom second shortcut must be created. If the normal shortcut is used, ControllerBuddy will launch but the overlay will not be visible.

1. Create a new script file `launch_fighters_anthology.sh` in your home directory with the following content:
    ```bash
    #!/bin/bash

    # the variable app_id must be adjusted accordingly
    app_id=<APP_ID>

    export STEAM_COMPAT_CLIENT_INSTALL_PATH="$HOME/.steam/root/"
    export STEAM_COMPAT_DATA_PATH="$HOME/.local/share/Steam/steamapps/compatdata/$app_id"

    flatpak run de.bwravencl.ControllerBuddy -autostart local -profile  /app/share/ControllerBuddy-Profiles/Fighters_Anthology.json -tray &

    timeout 10 bash -c 'until grep -q "ControllerBuddy Joystick" /proc/bus/input/devices ; do sleep 1 ; done'
    exit_code=$?

    if ! "$HOME/.local/share/Steam/ubuntu12_32/steam-launch-wrapper" -- \
        "$HOME/.local/share/Steam/ubuntu12_32/reaper" SteamLaunch AppId="$app_id" -- \
        "$HOME/.local/share/Steam/steamapps/common/SteamLinuxRuntime_sniper/_v2-entry-point" --verb=waitforexitandrun -- \
        "$HOME/.local/share/Steam/steamapps/common/Proton 9.0 (Beta)/proton" waitforexitandrun \
        "$HOME/.local/share/Steam/steamapps/compatdata/$app_id/pfx/drive_c/JANES/Fighters Anthology/FA.EXE"
    then
        if [ $exit_code -eq 124 ]; then
            zenity --error --text="Launch aborted because ControllerBuddy wasn't ready within 10 seconds.\n\nCheck if your controller is connected." --width 500
        fi
    fi

    killall -q ControllerBuddy
    ```
2. Replace the placeholder `<APP_ID>` in the script with the actual **APP ID** obtained in step 6 of the main guide.
3. Make the script executable:
    ```sh
    chmod +x "$HOME/launch_fighters_anthology.sh"
    ```
4. Add the script as a Non-Steam game in your Steam library.
5. Rename the **launch_fighters_anthology.sh** Steam shortcut to **Jane's Fighters Anthology (Gaming Mode)**.

> [!IMPORTANT]
> The other Steam shortcut must not be deleted, as this would also delete the Proton prefix.

### Configure Touchpads

There is a special ControllerBuddy Steam Input controller layout available which configures the Steam Deck's touchpads to act as a mouse replacement.

| Control              | Function           |
|----------------------|--------------------|
| Right Touchpad       | Move mouse cursor  |
| Right Touchpad Click | Left mouse button  |
| Left Touchpad Click  | Right mouse button |
| Left Touchpad Y-Axis | Scroll up/down     |

To this the layout:
1. Click [here](steam://controllerconfig/25189661165/3605812061) to obtain the controller layout.
2. Apply the layout to both **Jane's Fighters Anthology** shortcuts in your Steam library.
