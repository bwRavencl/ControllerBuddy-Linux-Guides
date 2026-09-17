<!-- markdownlint-disable-file line-length -->
# IL-2 Sturmovik: Great Battles with ControllerBuddy on Linux

## 📖 Description

This guide describes the installation of [IL-2 Sturmovik: Great Battles](https://il2sturmovik.com) (Steam) for use with [ControllerBuddy](https://controllerbuddy.org) on Linux via the [Proton](https://github.com/ValveSoftware/Proton) compatibility layer.

What you get with this setup:

- An installation of IL-2 Sturmovik: Great Battles that is nicely integrated into your Steam library.
- ControllerBuddy will start automatically when you start the game, load the correct profile, and exit when you quit the game.

## 🧩 Prerequisites

- [Steam](https://steampowered.com) (distribution package)
- [ControllerBuddy Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak)
- [IL-2 Sturmovik: Battle of Stalingrad Steam Release](https://store.steampowered.com/app/307960/IL2_Sturmovik_Battle_of_Stalingrad)

## 🪜 Steps

> [!IMPORTANT]
> Before starting with the steps, make sure to read the [Important Notes](README.md#%EF%B8%8F-important-notes) section in the [README](README.md) of this repository.

1. Download IL-2 Sturmovik: Battle of Stalingrad in Steam.

1. Select **Proton 11.0** as compatibility tool.

1. Update the **IL-2 Sturmovik: Battle of Stalingrad** Steam shortcut as follows:

    **Launch Options**:

    ```text
    "$("$STEAM_RUNTIME"/scripts/switch-runtime.sh --runtime='' -- flatpak info -l de.bwravencl.ControllerBuddy)/files/share/proton-wrapper.sh" IL-2_GB %command%
    ```

1. If you are using a controller that requires Steam Input, select the **Gamepad With Camera Controls** layout for **IL-2 Sturmovik: Battle of Stalingrad** to ensure the controller will be detected by ControllerBuddy.  
In case of the Steam Deck, apply the special ControllerBuddy layout instead as described in the [Steam Deck Specifics](README.md#%EF%B8%8F-touchpad-configuration) section below.
