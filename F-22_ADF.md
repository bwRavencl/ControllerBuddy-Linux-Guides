# F-22: Air Dominance Fighter with ControllerBuddy on Linux

## 📖 Description

This guide describes the installation of [F-22: Air Dominance Fighter](https://en.wikipedia.org/wiki/F-22:_Air_Dominance_Fighter) (Steam) for use with [ControllerBuddy](https://controllerbuddy.org) on Linux via the [Proton](https://github.com/ValveSoftware/Proton) compatibility layer.

What you get with this setup:
- An installation of F-22: Air Dominance Fighter that is nicely integrated into your Steam library.
- ControllerBuddy will start automatically when you start the game, load the correct profile, and exit when you quit the game.

## 🧩 Prerequisites

- [Steam](https://steampowered.com) (distribution package)
- [protontricks Flatpak](https://flathub.org/en/apps/com.github.Matoking.protontricks)
- [ControllerBuddy Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak)
- [F-22: Air Dominance Fighter Steam Release](https://store.steampowered.com/app/3146140/F22_Air_Dominance_Fighter)

## 🪜 Steps

1. Download F-22: Air Dominance Fighter in Steam.

2. Select **Proton 10.0-4** as compatibility tool.

3. Launch and immediately exit **F-22: Air Dominance Fighter** so that the Proton prefix gets created and the registry entries of F-22: Air Dominance Fighter get created.

4. Export the `APP_ID` environment variable:
    ```sh
    export APP_ID=3146140
    ```

> [!IMPORTANT]
> All subsequent commands must be executed within the same shell session to retain the `APP_ID` environment variable.

5. Make sure all your game controllers are connected.

6. Hide all game controllers from the Proton prefix, except for ControllerBuddy's UINPUT joystick device:
    ```sh
    reg_file=$(mktemp -p '' joysticks-XXXX.reg) &&
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

7. Install **directplay** into the Proton prefix:
    ```sh
    flatpak run com.github.Matoking.protontricks "$APP_ID" directplay
    ```

8. Update the **F-22: Air Dominance Fighter** Steam shortcut as follows:

    **LAUNCH OPTIONS**:
    ```sh
    ${STEAM_RUNTIME}/scripts/switch-runtime.sh --runtime='' -- flatpak run de.bwravencl.ControllerBuddy -autostart local -profile /app/share/ControllerBuddy-Profiles/F-22_ADF.json -tray & timeout 10 bash -c 'until grep -q "ControllerBuddy Joystick" /proc/bus/input/devices ; do sleep 1 ; done' && %command% || { [ $? -eq 124 ] && zenity --error --text="Launch aborted because ControllerBuddy wasn't ready within 10 seconds.\n\nCheck if your controller is connected." --width 500 ; } ; killall -q ControllerBuddy
    ```

## 🎮 Steam Deck Specifics

### Launching from Gaming Mode

To allow launching F-22: Air Dominance Fighter with ControllerBuddy from the Steam Deck's Gaming Mode, a custom second shortcut must be created. If the normal shortcut is used, ControllerBuddy will launch but the overlay will not be visible.

1. Create a new script file `F-22_ADF.sh` in your home directory:
    ```sh
    cat << 'EOF' > "$HOME/F-22_ADF.sh"
    #!/bin/bash
    app_id=3146140

    export STEAM_COMPAT_CLIENT_INSTALL_PATH="$HOME/.steam/root/"
    export STEAM_COMPAT_DATA_PATH="$HOME/.local/share/Steam/steamapps/compatdata/$app_id"

    flatpak run de.bwravencl.ControllerBuddy -autostart local -profile /app/share/ControllerBuddy-Profiles/F-22_ADF.json -tray &

    timeout 10 bash -c 'until grep -q "ControllerBuddy Joystick" /proc/bus/input/devices ; do sleep 1 ; done'
    if [ "$?" -eq 124 ]; then
        zenity --error --text="Launch aborted because ControllerBuddy wasn't ready within 10 seconds.\n\nCheck if your controller is connected." --width 500
    fi

    "$HOME/.local/share/Steam/ubuntu12_32/steam-launch-wrapper" -- \
        "$HOME/.local/share/Steam/ubuntu12_32/reaper" SteamLaunch AppId="$app_id" -- \
        "$HOME/.local/share/Steam/steamapps/common/SteamLinuxRuntime_sniper/_v2-entry-point" --verb=waitforexitandrun -- \
        "$HOME/.local/share/Steam/steamapps/common/Proton 10.0/proton" waitforexitandrun \
        "$HOME/.local/share/Steam/steamapps/common/F22ADF/adfusa.exe"

    killall -q ControllerBuddy

    EOF
    ```

2. Make the script executable:
    ```sh
    chmod +x "$HOME/F-22_ADF.sh"
    ```

3. Add the script as a Non-Steam game to your Steam library.

4. Rename the **F-22_ADF.sh** Steam shortcut to **F-22: Air Dominance Fighter (Gaming Mode)**.

### Configure Touchpads

There is a special ControllerBuddy Steam Input controller layout available which configures the Steam Deck's touchpads to act as a mouse replacement.

| Control              | Function           |
|----------------------|--------------------|
| Right Touchpad       | Move mouse cursor  |
| Right Touchpad Click | Left mouse button  |
| Left Touchpad Click  | Right mouse button |
| Left Touchpad Y-Axis | Scroll up/down     |

To use this layout:

1. Open the following link in the Steam browser to obtain the controller layout: `steam://controllerconfig/25189661165/3605812061`

2. Apply the layout to both **F-22: Air Dominance Fighter** shortcuts in your Steam library.
