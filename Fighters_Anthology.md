<!-- markdownlint-disable-file line-length -->
# Jane's Fighters Anthology with ControllerBuddy on Linux

## 📖 Description

This guide describes the installation of [Jane's Fighters Anthology](https://en.wikipedia.org/wiki/Fighters_Anthology) for use with [ControllerBuddy](https://controllerbuddy.org) on Linux via the [Proton](https://github.com/ValveSoftware/Proton) compatibility layer.

What you get with this setup:

- An installation of Jane's Fighters Anthology that is nicely integrated into your Steam library.
- ControllerBuddy will start automatically when you start the game, load the correct profile, and exit when you quit the game.

## 🧩 Prerequisites

- [Steam](https://steampowered.com) (distribution package)
- [ControllerBuddy Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak)
- [Protontricks Flatpak](https://flathub.org/en/apps/com.github.Matoking.protontricks)
- `FA_1_00F.iso` (Jane's Fighters Anthology installation disc ISO image)

## 🪜 Steps

> [!IMPORTANT]
> Before starting with the steps, make sure to read the [Important Notes](README.md#%EF%B8%8F-important-notes) section in the [README](README.md) of this repository.

1. Mount the `FA_1_00F.iso` disc image to a temporary directory:

    ```sh
    export CDROM_DIR=$(mktemp -d cdrom-XXXX) &&
    sudo mount -o loop FA_1_00F.iso "$CDROM_DIR" &&
    echo "setup.exe located at: $CDROM_DIR/setup.exe"
    ```

> [!IMPORTANT]
> All subsequent commands must be executed within the same shell session to retain the `CDROM_DIR` environment variable.

1. Add `setup.exe` as a Non-Steam game.

1. Rename the **setup.exe** Steam shortcut to **Jane's Fighters Anthology**.

1. Select **Proton 11.0** as compatibility tool.

1. Launch the **Jane's Fighters Anthology** Steam shortcut and install Jane's Fighters Anthology.  
    During the setup, select "Full Install - Digital Music".

1. Obtain the `APP_ID` of the Proton prefix:

    ```sh
    export APP_ID=$(flatpak run com.github.Matoking.protontricks -l \
        | grep "^Non-Steam shortcut: Jane's Fighters Anthology ([0-9]\+)$" \
        | sed -E 's/.*\(([0-9]+)\).*/\1/' \
        | head -n1)
    echo "APP ID: $APP_ID"
    ```

> [!IMPORTANT]
> All subsequent commands must be executed within the same shell session to retain the `APP_ID` environment variable.

1. Download `fae102.exe` with your browser:

    ```sh
    xdg-open https://jkpeterson.net/fa/misc/fae102.exe &
    ```

1. Install the `fae102.exe` patch using Protontricks:

    ```sh
    cd $(xdg-user-dir DOWNLOAD) &&
    flatpak run com.github.Matoking.protontricks -c 'wine fae102.exe' "$APP_ID"
    ```

> [!IMPORTANT]
> During the patch installation, `C:\JANES\Fighters Anthology\FA.EXE` must be selected via **My Computer**.

1. Download the `FAno-cd.zip` no-disc patch with your browser:

    ```sh
    xdg-open https://jkpeterson.net/fa/misc/FAno-cd.zip &
    ```

1. Extract the `FAno-cd.zip` no-disc patch to your Fighters Anthology installation directory:

    ```sh
    cd $(xdg-user-dir DOWNLOAD) &&
    unzip -o FAno-cd.zip -x README.txt -d "$HOME/.local/share/Steam/steamapps/compatdata/$APP_ID/pfx/drive_c/JANES/Fighters Anthology"
    ```

1. Replace the fake `fa_7.lib` of the no-disc patch with `fa_7.lib` from the disc image:

    ```sh
    cp "$CDROM_DIR/fa_7.lib" "$HOME/.local/share/Steam/steamapps/compatdata/$APP_ID/pfx/drive_c/JANES/Fighters Anthology/"
    ```

> [!IMPORTANT]
> Do not replace the fake `fa_4c.lib` with the one from the disc, as it will cause a crash to desktop during startup.

1. Unmount the `FA_1_00F.iso` file and delete the temporary directory:

    ```sh
    sudo umount "$CDROM_DIR" &&
    rm -rf "$CDROM_DIR"
    ```

> [!IMPORTANT]
> In the following step the placeholders denoted by `<...>` must be replaced accordingly to the following table:
>
> | Placeholder | Description                                     |
> |-------------|-------------------------------------------------|
> | `<USER>`    | Your username                                   |
> | `<APP_ID>`  | The Proton prefix **APP ID** obtained in step 6 |

1. Update the **Jane's Fighters Anthology** Steam shortcut as follows:

    **TARGET**:

    ```text
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/JANES/Fighters Anthology/FA.EXE"
    ```

    **START IN**:

    ```text
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/JANES/Fighters Anthology"
    ```

    **LAUNCH OPTIONS**:

    ```sh
    "$("$STEAM_RUNTIME"/scripts/switch-runtime.sh --runtime='' -- flatpak info -l de.bwravencl.ControllerBuddy)/files/share/proton-wrapper.sh" Fighters_Anthology %command%
    ```

1. If you are using a controller that requires Steam Input, select the **Gamepad With Camera Controls** layout for **Jane's Fighters Anthology** to ensure the controller will be detected by ControllerBuddy.  
In case of the Steam Deck, apply the special ControllerBuddy layout instead as described in the [Steam Deck Specifics](README.md#%EF%B8%8F-touchpad-configuration) section below.
