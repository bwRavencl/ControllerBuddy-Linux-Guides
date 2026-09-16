<!-- markdownlint-disable-file line-length -->
# DCS World with ControllerBuddy on Linux

## 📖 Description

This guide describes the installation of [DCS World](https://www.digitalcombatsimulator.com) (standalone) for use with [ControllerBuddy](https://controllerbuddy.org) on Linux via the [Proton](https://github.com/ValveSoftware/Proton) compatibility layer.

What you get with this setup:

- A standalone installation of DCS World that is nicely integrated into your Steam library and configured for use with ControllerBuddy.
- ControllerBuddy will start automatically when you start the game and exit when you quit the game.
- When switching aircraft in DCS, ControllerBuddy will automatically load the corresponding profile.

## 🧩 Prerequisites

- [Steam](https://steampowered.com) (distribution package)
- [Git](https://git-scm.com) (distribution package)
- [ImageMagick](https://imagemagick.org) (distribution package)
- [ControllerBuddy Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak)

## 🪜 Steps

> [!IMPORTANT]
> Before starting with the steps, make sure to read the [Important Notes](README.md#%EF%B8%8F-important-notes) section in the [README](README.md) of this repository.

1. Download the DCS World installer (`DCS_World_web.exe`) from the [official download page](https://www.digitalcombatsimulator.com/en/downloads/world/stable).

1. Add `DCS_World_web.exe` as a Non-Steam game.

1. Rename the **DCS_World_web.exe** Steam shortcut to **DCS World**.

1. Select **Proton Experimental** as compatibility tool.

1. Launch the **DCS World** Steam shortcut and install DCS World.

1. Obtain the `APP_ID` of the Proton prefix:

    ```sh
    export APP_ID=$(flatpak run com.github.Matoking.protontricks -l \
        | grep '^Non-Steam shortcut: DCS World ([0-9]\+)$' \
        | sed -E 's/.*\(([0-9]+)\).*/\1/' \
        | head -n1)
    echo "APP ID: $APP_ID"
    ```

> [!IMPORTANT]
> All subsequent commands must be executed within the same shell session to retain the `APP_ID` environment variable.

1. Install `SEGUISYM.TTF` into the Proton prefix:

    ```sh
    cd "~/.local/share/Steam/steamapps/compatdata/$APP_ID/pfx/drive_c/windows/Fonts" &&
    curl -O -L https://raw.githubusercontent.com/microsoft/elfie-arriba/master/Arriba/Arriba.Web/fonts/SEGUISYM.TTF
    ```

> [!IMPORTANT]
> In the following step the placeholders denoted by `<...>` must be replaced accordingly to the following table:
>
> | Placeholder | Description                                     |
> |-------------|-------------------------------------------------|
> | `<USER>`    | Your username                                   |
> | `<APP_ID>`  | The Proton prefix **APP ID** obtained in step 6 |

1. Update the **DCS World** Steam shortcut as follows:

    **TARGET**:

    ```text
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/Program Files/Eagle Dynamics/DCS World/bin/DCS_updater.exe"
    ```

    **START IN**:

    ```text
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/Program Files/Eagle Dynamics/DCS World"
    ```

    **LAUNCH OPTIONS**:

    ```sh
    override_vram_size=8000 CONTROLLER_BUDDY_PROFILES_DIR=/app/share/ControllerBuddy-Profiles WINE_SIMULATE_WRITECOPY=1 WINEDLLOVERRIDES='wbemprox=n' "$("$STEAM_RUNTIME"/scripts/switch-runtime.sh --runtime='' -- flatpak info -l de.bwravencl.ControllerBuddy)/files/share/proton-wrapper.sh" DCS_Su-27 %command%
    ```

> [!NOTE]
> The above launch options use `override_vram_size=8000` to prevent DCS from overallocating VRAM.  
> This variable only affects Mesa GPU drivers.  
> The 8000 MB limit is optimized for a 16 GB VRAM card - scale this number up or down to match your own graphics card's memory.

1. If you are using a controller that requires Steam Input, select the **Gamepad With Camera Controls** layout for **DCS World** to ensure the controller will be detected by ControllerBuddy.

1. Launch the **DCS World** Steam shortcut to download and install your modules.

1. Convert the `DCS-1.ico` file to `.png`:

    ```sh
    ICONS_DIR="$(xdg-user-dir PICTURES)/Icons"
    cd "$HOME/.local/share/Steam/steamapps/compatdata/$APP_ID/pfx/drive_c/Program Files/Eagle Dynamics/DCS World/FUI" &&
    mkdir -p "$ICONS_DIR" &&
    magick DCS-1.ico "$ICONS_DIR/DCS_World.png"
    ```

1. Edit the **DCS World** Steam shortcut and select `/home/<USER>/<PICTURES_DIR>/Icons/DCS_World.png` as the icon.

1. Set up [ControllerBuddy-DCS-Integration](https://github.com/bwRavencl/ControllerBuddy-DCS-Integration):

    ```sh
    scripts_dir="~/.local/share/Steam/steamapps/compatdata/$APP_ID/pfx/drive_c/users/steamuser/Saved\ Games/DCS/Scripts" &&
    git clone https://github.com/bwRavencl/ControllerBuddy-DCS-Integration.git "$scripts_dir/ControllerBuddy-DCS-Integration" &&
    echo 'dofile(lfs.writedir()..[[Scripts\ControllerBuddy-DCS-Integration\ControllerBuddy.lua]])' > "$scripts_dir/Export.lua"
    ```
