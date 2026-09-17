<!-- markdownlint-disable-file line-length -->
# Falcon 3.0 with ControllerBuddy on Linux

## 📖 Description

This guide describes the setup of [Falcon 3.0](https://en.wikipedia.org/wiki/Falcon_3.0) (Steam) for use with [ControllerBuddy](https://controllerbuddy.org) on Linux via the [DOSBox Staging](https://www.dosbox-staging.org) emulator.

What you get with this setup:

- An installation of Falcon 3.0 that is nicely integrated into your Steam library.
- ControllerBuddy will start automatically when you start the game, load the correct profile, and exit when you quit the game.

## 🧩 Prerequisites

- [Steam](https://steampowered.com) (distribution package)
- [ControllerBuddy Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak)
- [Falcon Gold Steam Release](https://store.steampowered.com/app/429520/Falcon_Gold)

## 🪜 Steps

> [!IMPORTANT]
> Before starting with the steps, make sure to read the [Important Notes](README.md#%EF%B8%8F-important-notes) section in the [README](README.md) of this repository.

1. Download Falcon Gold in Steam.

1. Create a folder which will hold a custom DOSBox config file:

    ```sh
    mkdir "$HOME/Games/Falcon_Gold"
    ```

1. Create the DOSBox config file:

    ```sh
    cat << 'EOF' > "$HOME/Games/Falcon_Gold/dosbox_falcon_gold.conf"
    [sdl]
    fullscreen = true
    host_rate = sdi
    presentation_mode = cfr

    [dosbox]
    memsize = 8
    vmem_delay = 650

    [render]
    glshader = nearest

    [cpu]
    cpu_cycles = 20000
    cpu_cycles_protected = auto
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
    mount c "~/.local/share/Steam/steamapps/common/Falcon Gold/game"
    imgmount d "~/.local/share/Steam/steamapps/common/Falcon Gold/game/game.gld" -t iso -fs iso
    c:
    cls
    FALCONCD.EXE
    exit

    EOF
    ```

> [!IMPORTANT]
> The launch command in the following step ends with an optional argument `[mouse_sensitivity]`.
> You can simply omit this argument, but if your mouse feels too sensitive, try appending a value like `0.3` to the command.

1. Update the **Falcon Gold** Steam shortcut as follows:

    **Launch Options**:

    ```text
    IGNORE='%command%'; "$("$STEAM_RUNTIME"/scripts/switch-runtime.sh --runtime='' -- flatpak info -l de.bwravencl.ControllerBuddy)/files/share/dosbox-wrapper.sh" Falcon_3.0 "$HOME/Games/Falcon_Gold/dosbox_falcon_gold.conf" [mouse_sensitivity]
    ```

1. If you are using a controller that requires Steam Input, select the **Gamepad With Camera Controls** layout for **Falcon Gold** to ensure the controller will be detected by ControllerBuddy.  
In case of the Steam Deck, apply the special ControllerBuddy layout instead as described in the [Steam Deck Specifics](README.md#%EF%B8%8F-touchpad-configuration) section below.
