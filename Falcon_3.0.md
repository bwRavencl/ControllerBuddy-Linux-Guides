# Falcon 3.0 with ControllerBuddy on Linux

## 📖 Description

This guide describes the setup of [Falcon 3.0](https://en.wikipedia.org/wiki/Falcon_3.0) (Steam) for use with [ControllerBuddy](https://controllerbuddy.org/) on Linux via the [DOSBox Staging](https://www.dosbox-staging.org) emulator.

What you get with this setup:
- An installation of Falcon 3.0 that is nicely integrated into your Steam library.
- ControllerBuddy will start automatically when you start the game, load the correct profile, and exit after you exit the game.

## 🧩 Prerequisites

- [Steam](https://steampowered.com/) (distribution package)
- [DOSBox Staging Flatpak](https://flathub.org/en/apps/io.github.dosbox-staging)
- [ControllerBuddy Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak)
- [Falcon Gold Steam Release](https://store.steampowered.com/app/429520/Falcon_Gold)

## 🪜 Steps

1. Download Falcon Gold in Steam.
2. Create a folder which will hold a custom DOSBox config file and launch script for Falcon 3.0.
    ```sh
    mkdir "$HOME/Falcon_Gold"
    ```
3. Configure Falcon Gold to work with the `Falcon_3.0.json` profile from [ControllerBuddy-Profiles](https://github.com/bwRavencl/ControllerBuddy-Profiles):
    ```sh
    controller_buddy_profiles_dir=$(realpath -s "$(flatpak info -l de.bwravencl.ControllerBuddy)/../active/files/share/ControllerBuddy-Profiles") &&
    cp "$controller_buddy_profiles_dir/configs/Falcon_3.0/JOYSTICK.DEF" "$HOME/.local/share/Steam/steamapps/common/Falcon Gold/game/"
    ```
4. Create the DOSBox config file.
    ```sh
    cat << 'EOF' > "$HOME/Falcon_Gold/dosbox_falcon_gold.conf"
    [sdl]
    fullscreen = true

    [dosbox]
    memsize = 8

    [render]
    glshader = nearest

    [cpu]
    cpu_cycles = 18000
    cycleup = 1000
    cycledown = 1000

    [voodoo]
    voodoo = false

    [joystick]
    joysticktype = 4axis
    timed = false
    deadzone = 0

    [dos]
    keyboardlayout = us

    [ipx]
    ipx = true

    [autoexec]
    @echo off
    mount c "/home/matteo/.local/share/Steam/steamapps/common/Falcon Gold/game"
    imgmount d "/home/matteo/.local/share/Steam/steamapps/common/Falcon Gold/game/game.gld" -t iso -fs iso
    c:
    cls
    FALCONCD.EXE
    exit
    EOF
    ```
5. Create the launch script.
    ```sh
    cat << 'EOF' > "$HOME/Falcon_Gold/Falcon_Gold.sh"
    #!/bin/bash

    CB_PROFILE=Falcon_3.0.json
    DOSBOX_CONF=dosbox_falcon_gold.conf

    cd "$(cd -- "$(dirname -- "$0")" &> /dev/null && pwd)" || exit

    flatpak run de.bwravencl.ControllerBuddy -autostart local -profile "/app/share/ControllerBuddy-Profiles/$CB_PROFILE" -tray &

    WAIT_TIMEOUT=10
    CB_DEVICE_NAME="ControllerBuddy Joystick"

    for (( i=1; i<=WAIT_TIMEOUT; i++ )); do
        CB_JOYSTICK_DEVICE=$(awk -v RS='' "/Name=\"$CB_DEVICE_NAME\"/{match(\$0, /js[0-9]+/); print substr(\$0, RSTART, RLENGTH); exit}" /proc/bus/input/devices)

        if [ -n "$CB_JOYSTICK_DEVICE" ]; then
            break
        fi

        sleep 1
    done

    function quit_cb {
        killall -q ControllerBuddy
    }

    if [ -z "$CB_JOYSTICK_DEVICE" ]; then
        zenity --error --text="Launch aborted because $CB_DEVICE_NAME wasn't ready within $WAIT_TIMEOUT seconds.\n\nCheck if your controller is connected." --width 500
        quit_cb
        exit 1
    fi

    SDL_JOYSTICK_DEVICE="/dev/input/$CB_JOYSTICK_DEVICE" \
    SDL_MOUSE_RELATIVE_SPEED_SCALE=0.3 \
    flatpak run io.github.dosbox-staging -conf "$DOSBOX_CONF"

    quit_cb

    EOF
    ```
6. Add the launch script (`$HOME/Falcon_Gold/Falcon_Gold.sh`) as a Non-Steam game to your Steam library.
7. Rename the **Falcon_Gold.sh** Steam shortcut to **Falcon Gold (ControllerBuddy)**.

## 🎮 Steam Deck Specifics

### Configure Touchpads

There is a special ControllerBuddy Steam Input controller layout available which configures the Steam Deck's touchpads to act as a mouse replacement.

| Control              | Function           |
|----------------------|--------------------|
| Right Touchpad       | Move mouse cursor  |
| Right Touchpad Click | Left mouse button  |
| Left Touchpad Click  | Right mouse button |
| Left Touchpad Y-Axis | Scroll up/down     |

To use this the layout:
1. Open the following link in the Steam browser to obtain the controller layout: `steam://controllerconfig/25189661165/3605812061`
2. Apply the layout to both **Jane's Fighters Anthology** shortcuts to your Steam library.
