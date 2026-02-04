# DCS World with ControllerBuddy on Linux

## 📖 Description

This guide describes the installation of [DCS World](https://www.digitalcombatsimulator.com) (standalone) for use with [ControllerBuddy](https://controllerbuddy.org) on Linux via the [Proton](https://github.com/ValveSoftware/Proton) compatibility layer.

What you get with this setup:
- A standalone installation of DCS World that is nicely integrated into your Steam library and configured for use with ControllerBuddy.
- ControllerBuddy will start automatically when you start the game and exit after you exit the game.
- When switching aircraft in DCS, ControllerBuddy will automatically load the corresponding profile.

## 🧩 Prerequisites

- [Steam](https://steampowered.com) (distribution package)
- [Git](https://git-scm.com) (distribution package)
- [ImageMagick](https://imagemagick.org) (distribution package)
- [GE-Proton-10-27](https://github.com/GloriousEggroll/proton-ge-custom/releases/tag/GE-Proton10-26)
- [protontricks Flatpak](https://flathub.org/en/apps/com.github.Matoking.protontricks)
- [ControllerBuddy Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak)

## 🪜 Steps

1. Download the DCS World installer (`DCS_World_web.exe`) from [here](https://www.digitalcombatsimulator.com/en/downloads/world/stable).

2. Add `DCS_World_web.exe` as a Non-Steam game.

3. Rename the **DCS_World_web.exe** Steam shortcut to **DCS World**.

4. Select **GE-Proton-10-27** as compatibility tool.

5. Launch the **DCS World** Steam shortcut and install DCS.

6. Obtain the `APP_ID` of the Proton prefix:
    ```sh
    export APP_ID=$(flatpak run com.github.Matoking.protontricks -l \
        | grep '^Non-Steam shortcut: DCS World ([0-9]\+)$' \
        | sed -E 's/.*\(([0-9]+)\).*/\1/' \
        | head -n1)
    echo "APP ID: $APP_ID"
    ```

> [!IMPORTANT]
> All subsequent commands must be executed within the same shell session to retain the `APP_ID` environment variable.

7. Install **d3dcompiler_47** into the Proton prefix:
    ```sh
    flatpak run com.github.Matoking.protontricks "$APP_ID" d3dcompiler_47 vcrun2022
    ```

8. Make sure all your game controllers are connected.

9. Hide all game controllers from the Proton prefix, except for ControllerBuddy's UINPUT joystick device:
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

10. Install `SEGUISYM.TTF` into the Proton prefix:
    ```sh
    cd "~/.local/share/Steam/steamapps/compatdata/$APP_ID/pfx/drive_c/windows/Fonts" &&
    curl -O -L https://raw.githubusercontent.com/microsoft/elfie-arriba/master/Arriba/Arriba.Web/fonts/SEGUISYM.TTF
    ```

> [!IMPORTANT]
> In the following step the placeholders denoted by `<...>` must be replaced accordingly to the following table:
> | Placeholder | Description                                     |
> |-------------|-------------------------------------------------|
> | `<USER>`    | Your username                                   |
> | `<APP_ID>`  | The Proton prefix **APP ID** obtained in step 6 |

11. Update the **DCS World** Steam shortcut as follows:

    **TARGET**:
    ```
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/Program Files/Eagle Dynamics/DCS World/bin/DCS_updater.exe"
    ```

    **START IN**:
    ```
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/Program Files/Eagle Dynamics/DCS World"
    ```

    **LAUNCH OPTIONS**:
    ```sh
    ${STEAM_RUNTIME}/scripts/switch-runtime.sh --runtime='' -- flatpak run de.bwravencl.ControllerBuddy -autostart local -tray & timeout 10 bash -c 'until grep -q "ControllerBuddy Joystick" /proc/bus/input/devices ; do sleep 1 ; done' && override_vram_size=8192 CONTROLLER_BUDDY_PROFILES_DIR=/app/share/ControllerBuddy-Profiles WINE_SIMULATE_WRITECOPY=1 WINEDLLOVERRIDES='wbemprox=n' %command% || { [ $? -eq 124 ] && zenity --error --text="Launch aborted because ControllerBuddy wasn't ready within 10 seconds.\n\nCheck if your controller is connected." --width 500 ; } ; killall -q ControllerBuddy
    ```

12. Launch the **DCS World** Steam shortcut to download and install your modules.

13. Convert the `DCS-1.ico` file to `.png`:
    ```sh
    ICONS_DIR="$(xdg-user-dir PICTURES)/Icons"
    cd "$HOME/.local/share/Steam/steamapps/compatdata/$APP_ID/pfx/drive_c/Program Files/Eagle Dynamics/DCS World/FUI" &&
    mkdir -p "$ICONS_DIR" &&
    magick DCS-1.ico "$ICONS_DIR/DCS_World.png"
    ```

14. Edit the **DCS World** Steam shortcut and select `/home/<USER>/<PICTURES_DIR>/Icons/DCS_World.png` as the icon.

15. Install [PowerShell 7+](https://github.com/PowerShell/PowerShell) into the Proton prefix:
    ```sh
    msi_file=$(mktemp -u PowerShell-XXXX.msi) &&
    curl -o "$msi_file" -L https://github.com/PowerShell/PowerShell/releases/download/v7.5.4/PowerShell-7.5.4-win-x64.msi &&
    flatpak run com.github.Matoking.protontricks -c "wine msiexec /i '$msi_file'" "$APP_ID"
    rm -f "$msi_file"
    ```

16. Set up [ControllerBuddy-DCS-Integration](https://github.com/bwRavencl/ControllerBuddy-DCS-Integration):
    ```sh
    scripts_dir="~/.local/share/Steam/steamapps/compatdata/$APP_ID/pfx/drive_c/users/steamuser/Saved\ Games/DCS/Scripts" &&
    git clone https://github.com/bwRavencl/ControllerBuddy-DCS-Integration.git "$scripts_dir/ControllerBuddy-DCS-Integration" &&
    echo 'dofile(lfs.writedir()..[[Scripts\ControllerBuddy-DCS-Integration\ControllerBuddy.lua]])' > "$scripts_dir/Export.lua"
    ```

17. Make sure your game controller is still connected.

18. Launch ControllerBuddy, and start local run mode to initialize the UINPUT joystick device:
    ```sh
    flatpak run de.bwravencl.ControllerBuddy -autostart local &
    ```

19. Configure DCS to work with the [ControllerBuddy-Profiles](https://github.com/bwRavencl/ControllerBuddy-Profiles):
    ```sh
    controller_buddy_profiles_dir=$(realpath -s "$(flatpak info -l de.bwravencl.ControllerBuddy)/../active/files/share/ControllerBuddy-Profiles") &&
    cd "$controller_buddy_profiles_dir/configs/DCS" &&
    WINEDEBUG='-all' flatpak run --filesystem="$controller_buddy_profiles_dir" com.github.Matoking.protontricks -c 'wine pwsh Configure.ps1' "$APP_ID"
    ```

## 🔄 Re-running the Configuration Script

The configuration script must be run again whenever the ControllerBuddy-Profiles receive an update.

1. Make sure your game controller is connected.

2. Execute the following command (steps 6, 18, and 19 combined):
    ```sh
    export APP_ID=$(flatpak run com.github.Matoking.protontricks -l \
        | grep '^Non-Steam shortcut: DCS World ([0-9]\+)$' \
        | sed -E 's/.*\(([0-9]+)\).*/\1/' \
        | head -n1)
    flatpak run de.bwravencl.ControllerBuddy -autostart local &
    controller_buddy_profiles_dir=$(realpath -s "$(flatpak info -l de.bwravencl.ControllerBuddy)/../active/files/share/ControllerBuddy-Profiles") &&
    cd "$controller_buddy_profiles_dir/configs/DCS" &&
    WINEDEBUG='-all' flatpak run --filesystem="$controller_buddy_profiles_dir" com.github.Matoking.protontricks -c 'wine pwsh Configure.ps1' "$APP_ID"
    ```
