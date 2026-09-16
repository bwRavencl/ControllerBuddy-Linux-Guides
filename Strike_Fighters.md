<!-- markdownlint-disable-file line-length -->
# Strike Fighters / Wings Over Vietnam / Wings Over Europe / Wings Over Israel with ControllerBuddy on Linux

## 📖 Description

This guide describes the installation of Strike Fighters, Wings Over Vietnam, Wings Over Europe, and Wings Over Israel for use with [ControllerBuddy](https://controllerbuddy.org) on Linux via the [Proton](https://github.com/ValveSoftware/Proton) compatibility layer.

What you get with this setup:

- An installation of your chosen game that is nicely integrated into your Steam library.
- ControllerBuddy will start automatically when you start the game, load the correct profile, and exit when you quit the game.

## 🧩 Prerequisites

- [Steam](https://steampowered.com) (distribution package)
- [Protontricks Flatpak](https://flathub.org/en/apps/com.github.Matoking.protontricks)
- [ControllerBuddy Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak)
- Game installation media or installer (`Setup.exe`)

## 🪜 Steps

> [!IMPORTANT]
> Before starting with the steps, make sure to read the [Important Notes](README.md#%EF%B8%8F-important-notes) section in the [README](README.md) of this repository.

1. Add `Setup.exe` as a Non-Steam game.

1. Rename the **setup.exe** Steam shortcut to **Strike Fighters**, **Wings Over Vietnam**, **Wings Over Europe**, or **Wings Over Israel** accordingly.

1. Select **Proton 9.0-4** as compatibility tool.

1. Launch the game Steam shortcut and install the game.

1. Obtain the `APP_ID` of the Proton prefix:

    ```sh
    flatpak run com.github.Matoking.protontricks -l
    ```

> [!IMPORTANT]
> Replace `<APP_ID>` with the actual APP ID obtained in this step.

1. Set the `APP_ID` variable:

    ```sh
    export APP_ID=<APP_ID>
    ```

> [!IMPORTANT]
> All subsequent commands must be executed within the same shell session to retain the `APP_ID` environment variable.
> Replace `<APP_ID>` with the actual APP ID obtained in this step.

1. Download and install the latest updates (Service Pack and 08.30.06 or September 2008 depending on the game) from the [Third Wire Download Archive](https://thirdwire.com/downloads_archive.htm):

    ```sh
    flatpak run com.github.Matoking.protontricks -c 'wine <UPDATER>.exe' "$APP_ID"
    ```

> [!IMPORTANT]
> In the following steps the placeholders denoted by `<...>` must be replaced accordingly to the following table:
>
> | Placeholder     | Description                                     |
> |-----------------|-------------------------------------------------|
> | `<USER>`        | Your username                                   |
> | `<APP_ID>`      | The Proton prefix **APP ID** obtained in step 5 |
> | `<GAME_FOLDER>` | The path to the game's installation folder      |
> | `<EXE>`         | The game's executable name                      |

1. Update the Steam shortcut as follows:

    **TARGET**:

    ```text
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/<GAME_FOLDER>/<EXE>.exe"
    ```

    **START IN**:

    ```text
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/<GAME_FOLDER>"
    ```

    **LAUNCH OPTIONS**:

    ```sh
    "$("$STEAM_RUNTIME"/scripts/switch-runtime.sh --runtime='' -- flatpak info -l de.bwravencl.ControllerBuddy)/files/share/proton-wrapper.sh" Strike_Fighters %command%
    ```

1. If you are using a controller that requires Steam Input, select the **Gamepad With Camera Controls** layout for **Strike Fighters** to ensure the controller will be detected by ControllerBuddy.  
In case of the Steam Deck, apply the special ControllerBuddy layout instead as described in the [Steam Deck Specifics](#-steam-deck-specifics) section below.

## 🎮 Steam Deck Specifics

### Configure Touchpads

> [!IMPORTANT]
> Since the Steam Deck's controller hardware is exposed to games via Steam Input, even if you do not care for the touchpad controls, you must at least apply the default Steam Input layout called **Gamepad With Camera Controls** to the **Strike Fighters** shortcut to ensure the controller can be detected by ControllerBuddy.

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

1. Apply the layout to the **Strike Fighters** shortcut in your Steam library.
