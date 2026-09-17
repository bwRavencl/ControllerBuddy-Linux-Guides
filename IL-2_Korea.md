<!-- markdownlint-disable-file line-length -->
# Korea. IL-2 Series with ControllerBuddy on Linux

## 📖 Description

This guide describes the installation of [Korea. IL-2 Series](https://il2-series.com) (Steam) for use with [ControllerBuddy](https://controllerbuddy.org) on Linux via the [Proton](https://github.com/ValveSoftware/Proton) compatibility layer.

What you get with this setup:

- An installation of Korea. IL-2 Series that is nicely integrated into your Steam library.
- ControllerBuddy will start automatically when you start the game, load the correct profile, and exit when you quit the game.

## 🧩 Prerequisites

- [Steam](https://steampowered.com) (distribution package)
- [ControllerBuddy Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak)
- [Korea. IL-2 Series Steam Release](https://store.steampowered.com/app/247970/Korea_IL2_Series/)

## 🪜 Steps

> [!IMPORTANT]
> Before starting with the steps, make sure to read the [Important Notes](README.md#%EF%B8%8F-important-notes) section in the [README](README.md) of this repository.

1. Download Korea. IL-2 Series in Steam.

1. Select **Proton Experimental** as compatibility tool.

1. Update the **Korea. IL-2 Series** Steam shortcut as follows:

    **Launch Options**:

    ```sh
   "$("$STEAM_RUNTIME"/scripts/switch-runtime.sh --runtime='' -- flatpak info -l de.bwravencl.ControllerBuddy)/files/share/proton-wrapper.sh" IL-2_Korea %command%
    ```

1. If you are using a controller that requires Steam Input, select the **Gamepad With Camera Controls** layout for **Korea. IL-2 Series** to ensure the controller will be detected by ControllerBuddy.  
In case of the Steam Deck, apply the special ControllerBuddy layout instead as described in the [Steam Deck Specifics](README.md#%EF%B8%8F-touchpad-configuration) section below.
