# IL-2 Sturmovik: Great Battles with ControllerBuddy on Linux

## 📖 Description

This guide describes the installation of [IL-2 Sturmovik: Great Battles](https://il2sturmovik.com) (Steam) for use with [ControllerBuddy](https://controllerbuddy.org) on Linux via the [Proton](https://github.com/ValveSoftware/Proton) compatibility layer.

What you get with this setup:
- An installation of IL-2 Sturmovik: Great Battles that is nicely integrated into your Steam library.
- ControllerBuddy will start automatically when you start the game, load the correct profile, and exit when you quit the game.

## 🧩 Prerequisites

- [Steam](https://steampowered.com) (distribution package)
- [protontricks Flatpak](https://flathub.org/en/apps/com.github.Matoking.protontricks)
- [ControllerBuddy Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak)
- [IL-2 Sturmovik: Great Battles Steam Release](https://store.steampowered.com/app/307960/IL2_Sturmovik_Battle_of_Stalingrad)

## 🪜 Steps

> [!IMPORTANT]
> Before starting with the steps, make sure to read the [Important Notes](README.md#%EF%B8%8F-important-notes) section in the [README](README.md) of this repository.

1. Download IL-2 Sturmovik: Great Battles in Steam.

2. Select **Proton 10.0-4** as compatibility tool.

3. Launch **IL-2 Sturmovik: Great Battles** so that the Proton prefix gets created. The game will likely crash on the first launch, but that's expected.

4. Export the `APP_ID` environment variable:
    ```sh
    export APP_ID=307960
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

7. Install **d3dcompiler_47** and **powershell** into the Proton prefix:
    ```sh
    flatpak run com.github.Matoking.protontricks "$APP_ID" d3dcompiler_47 powershell
    ```

8. Make sure your game controller is still connected.

9. Launch ControllerBuddy, and start local run mode to initialize the UINPUT joystick device:
    ```sh
    flatpak run de.bwravencl.ControllerBuddy -autostart local &
    ```

10. Configure IL-2 Sturmovik: Great Battles to work with the [ControllerBuddy-Profiles](https://github.com/bwRavencl/ControllerBuddy-Profiles):
    ```sh
    controller_buddy_profiles_dir=$(realpath -s "$(flatpak info -l de.bwravencl.ControllerBuddy)/../active/files/share/ControllerBuddy-Profiles") &&
    cd "$controller_buddy_profiles_dir/configs/IL-2_GB" &&
    WINEDEBUG='-all' flatpak run --filesystem="$controller_buddy_profiles_dir" com.github.Matoking.protontricks -c 'wine pwsh Configure.ps1' "$APP_ID"
    ```

11. Update the **IL-2 Sturmovik: Great Battles** Steam shortcut as follows:

    **LAUNCH OPTIONS**:
    ```sh
    ${STEAM_RUNTIME}/scripts/switch-runtime.sh --runtime='' -- flatpak run de.bwravencl.ControllerBuddy -autostart local -profile /app/share/ControllerBuddy-Profiles/IL-2_GB.json -tray & timeout 10 bash -c 'until grep -q "ControllerBuddy Joystick" /proc/bus/input/devices ; do sleep 1 ; done' && %command% || { [ $? -eq 124 ] && zenity --error --text="Launch aborted because ControllerBuddy wasn't ready within 10 seconds.\n\nCheck if your controller is connected." --width 500 ; } ; killall -q ControllerBuddy
    ```

## 🔄 Re-running the Configuration Script

The configuration script must be run again whenever the ControllerBuddy-Profiles receive an update.

1. Make sure your game controller is connected.

2. Execute the following command (steps 4, 9, and 10 combined):

    ```sh
    export APP_ID=307960
    flatpak run de.bwravencl.ControllerBuddy -autostart local &
    controller_buddy_profiles_dir=$(realpath -s "$(flatpak info -l de.bwravencl.ControllerBuddy)/../active/files/share/ControllerBuddy-Profiles") &&
    cd "$controller_buddy_profiles_dir/configs/IL-2_GB" &&
    WINEDEBUG='-all' flatpak run --filesystem="$controller_buddy_profiles_dir" com.github.Matoking.protontricks -c 'wine pwsh Configure.ps1' "$APP_ID"
    ```