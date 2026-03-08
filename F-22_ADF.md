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

> [!IMPORTANT]
> Before starting with the steps, make sure to read the [Important Notes](README.md#%EF%B8%8F-important-notes) section in the [README](README.md) of this repository.

1. Download F-22: Air Dominance Fighter in Steam.

2. Select **Proton 10.0-4** as compatibility tool.

3. Launch and immediately exit **F-22: Air Dominance Fighter** so that the Proton prefix gets created.

4. Export the `APP_ID` environment variable:
    ```sh
    export APP_ID=3146140
    ```

> [!IMPORTANT]
> All subsequent commands must be executed within the same shell session to retain the `APP_ID` environment variable.

5. Install **directplay** and **powershell** into the Proton prefix:
    ```sh
    flatpak run com.github.Matoking.protontricks "$APP_ID" directplay powershell
    ```

> [!IMPORTANT]
> If you are running the game on a Steam Deck, you can skip the remaining steps of this section, and instead follow the instructions in the [Steam Deck Specifics](#-steam-deck-specifics) section below to set up a special launch script and ControllerBuddy layout for the Steam Deck.

6. Make sure your game controller is still connected.

7. Launch ControllerBuddy, and start local run mode to initialize the UINPUT joystick device:
    ```sh
    flatpak run de.bwravencl.ControllerBuddy -autostart local &
    ```

8. Configure F-22: Air Dominance Fighter to work with the [ControllerBuddy-Profiles](https://github.com/bwRavencl/ControllerBuddy-Profiles):
    ```sh
    controller_buddy_profiles_dir=$(realpath -s "$(flatpak info -l de.bwravencl.ControllerBuddy)/../active/files/share/ControllerBuddy-Profiles") &&
    cd "$controller_buddy_profiles_dir/configs/F-22_ADF" &&
    WINEDEBUG='-all' flatpak run --filesystem="$controller_buddy_profiles_dir" com.github.Matoking.protontricks -c 'wine pwsh Configure.ps1' "$APP_ID"
    ```

9. Update the **F-22: Air Dominance Fighter** Steam shortcut as follows:

    **LAUNCH OPTIONS**:
    ```sh
    "${STEAM_RUNTIME}"/scripts/switch-runtime.sh --runtime='' -- flatpak run de.bwravencl.ControllerBuddy -autostart local -profile /app/share/ControllerBuddy-Profiles/F-22_ADF.json -tray & timeout=15; timeout "$timeout" bash -c 'until grep -q "ControllerBuddy Joystick" /proc/bus/input/devices ; do sleep 1 ; done' && %command% || { [ $? -eq 124 ] && zenity --error --text="Launch aborted because ControllerBuddy wasn't ready within $timeout seconds.\n\nCheck if your controller is connected." --width 500 ; } ; killall -q ControllerBuddy
    ```

## 🔄 Re-running the Configuration Script

The configuration script must be run again whenever the ControllerBuddy-Profiles receive an update.

1. Make sure your game controller is connected.

2. Execute the following command (steps 4, 7, and 8 combined):

    ```sh
    export APP_ID=3146140
    flatpak run de.bwravencl.ControllerBuddy -autostart local &
    controller_buddy_profiles_dir=$(realpath -s "$(flatpak info -l de.bwravencl.ControllerBuddy)/../active/files/share/ControllerBuddy-Profiles") &&
    cd "$controller_buddy_profiles_dir/configs/F-22_ADF" &&
    WINEDEBUG='-all' flatpak run --filesystem="$controller_buddy_profiles_dir" com.github.Matoking.protontricks -c 'wine pwsh Configure.ps1' "$APP_ID"
    ```

## 🎮 Steam Deck Specifics

### Launch Script

On the Steam Deck, a custom launch script is required to run ControllerBuddy alongside F-22: Air Dominance Fighter, since the configuration script needs to be run within the Steam Runtime environment to correctly determine the UUID of the ControllerBuddy UINPUT joystick device.

1. Create the `F-22_ADF.sh` launch script in your home directory:
    ```sh
    cat << 'EOF' > "$HOME/F-22_ADF.sh" && chmod +x "$HOME/F-22_ADF.sh"
    #!/bin/bash
    SteamAppId=3146140
    game_dir="$HOME/.local/share/Steam/steamapps/common/F22ADF"

    export STEAM_COMPAT_CLIENT_INSTALL_PATH="$HOME/.steam/root/"
    export STEAM_COMPAT_DATA_PATH="$HOME/.local/share/Steam/steamapps/compatdata/$SteamAppId"

    flatpak run de.bwravencl.ControllerBuddy -autostart local -profile /app/share/ControllerBuddy-Profiles/F-22_ADF.json -tray &

    timeout=15
    timeout "$timeout" bash -c 'until grep -q "ControllerBuddy Joystick" /proc/bus/input/devices ; do sleep 1 ; done'
    if [ "$?" -eq 124 ]
    then
        zenity --error --text="Launch aborted because ControllerBuddy wasn't ready within $timeout seconds.\n\nCheck if your controller is connected." --width 500
    else
        controller_buddy_profiles_dir=$(realpath -s "$(flatpak info -l de.bwravencl.ControllerBuddy)/../active/files/share/ControllerBuddy-Profiles") &&
        cd "$controller_buddy_profiles_dir/configs" &&
        "$HOME/.local/share/Steam/ubuntu12_32/steam-launch-wrapper" -- \
            "$HOME/.local/share/Steam/ubuntu12_32/reaper" SteamLaunch AppId="$SteamAppId" -- \
            "$HOME/.local/share/Steam/steamapps/common/SteamLinuxRuntime_sniper/_v2-entry-point" --verb=waitforexitandrun -- \
            "$HOME/.local/share/Steam/steamapps/common/Proton 10.0/proton" waitforexitandrun \
            pwsh F-22_ADF\\Configure.ps1 &&

        cd "$game_dir" &&
        "$HOME/.local/share/Steam/ubuntu12_32/steam-launch-wrapper" -- \
            "$HOME/.local/share/Steam/ubuntu12_32/reaper" SteamLaunch AppId="$SteamAppId" -- \
            "$HOME/.local/share/Steam/steamapps/common/SteamLinuxRuntime_sniper/_v2-entry-point" --verb=waitforexitandrun -- \
            "$HOME/.local/share/Steam/steamapps/common/Proton 10.0/proton" waitforexitandrun \
            "$game_dir/adfusa.exe"
    fi

    killall -q ControllerBuddy

    EOF
    ```

2. Add the `F-22_ADF.sh` launch script as a Non-Steam game to your Steam library.

3. Rename the **F-22_ADF.sh** Steam shortcut to **F-22: Air Dominance Fighter (ControllerBuddy)**.

### Configure Touchpads

> [!IMPORTANT]
> Since the Steam Deck's controller hardware is exposed to games via Steam Input, even if you do not care for the touchpad controls, you must at least apply the default Steam Input layout called **Gamepad With Camera Controls** to the **F-22: Air Dominance Fighter (ControllerBuddy)** shortcut to ensure the controller can be detected by ControllerBuddy.

There is a special ControllerBuddy Steam Input controller layout available which configures the Steam Deck's touchpads to act as a mouse replacement.

| Control              | Function           |
|----------------------|--------------------|
| Right Touchpad       | Move mouse cursor  |
| Right Touchpad Click | Left mouse button  |
| Left Touchpad Click  | Right mouse button |
| Left Touchpad Y-Axis | Scroll up/down     |

To use this layout:

1. Add the **ControllerBuddy** Steam Input layout to your Steam controller layouts:
    ```sh
    xdg-open steam://controllerconfig/3259858387/3672925155
    ```

2. Apply the **ControllerBuddy** layout to the **F-22: Air Dominance Fighter (ControllerBuddy)** shortcut in your Steam library.
