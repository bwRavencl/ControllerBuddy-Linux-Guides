<!-- markdownlint-disable-file line-length -->
# F-22: Air Dominance Fighter with ControllerBuddy on Linux

## 📖 Description

This guide describes the installation of [F-22: Air Dominance Fighter](https://en.wikipedia.org/wiki/F-22:_Air_Dominance_Fighter) (Steam) for use with [ControllerBuddy](https://controllerbuddy.org) on Linux via the [Proton](https://github.com/ValveSoftware/Proton) compatibility layer.

What you get with this setup:

- An installation of F-22: Air Dominance Fighter that is nicely integrated into your Steam library.
- ControllerBuddy will start automatically when you start the game, load the correct profile, and exit when you quit the game.

## 🧩 Prerequisites

- [Steam](https://steampowered.com) (distribution package)
- [ControllerBuddy Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak)
- [F-22: Air Dominance Fighter Steam Release](https://store.steampowered.com/app/3146140/F22_Air_Dominance_Fighter)

## 🪜 Steps

> [!IMPORTANT]
> Before starting with the steps, make sure to read the [Important Notes](README.md#%EF%B8%8F-important-notes) section in the [README](README.md) of this repository.

1. Download F-22: Air Dominance Fighter in Steam.

1. Select **Proton 11.0** as compatibility tool.

1. Update the **F-22: Air Dominance Fighter** Steam shortcut as follows:

    **Launch Options**:

    ```text
    "$("$STEAM_RUNTIME"/scripts/switch-runtime.sh --runtime='' -- flatpak info -l de.bwravencl.ControllerBuddy)/files/share/proton-wrapper.sh" F-22_ADF %command%
    ```

1. If you are using a controller that requires Steam Input, select the **Gamepad With Camera Controls** layout for **F-22: Air Dominance Fighter** to ensure the controller will be detected by ControllerBuddy.  
In case of the Steam Deck, apply the special ControllerBuddy layout instead as described in the [Steam Deck Specifics](README.md#%EF%B8%8F-touchpad-configuration) section below.
