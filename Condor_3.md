<!-- markdownlint-disable-file line-length -->
# Condor 3 with ControllerBuddy on Linux

## 📖 Description

This guide describes the installation of [Condor 3](https://www.condorsoaring.com/v3discover) for use with [ControllerBuddy](https://controllerbuddy.org) on Linux via the [Proton](https://github.com/ValveSoftware/Proton) compatibility layer.

What you get with this setup:

- An installation of Condor 3 that is nicely integrated into your Steam library.
- ControllerBuddy will start automatically when you start the game, load the correct profile, and exit when you quit the game.

## 🧩 Prerequisites

- [Steam](https://steampowered.com) (distribution package)
- [ControllerBuddy Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak)
- `CondorSetupV310.exe` (Condor 3 installer)

## 🪜 Steps

> [!IMPORTANT]
> Before starting with the steps, make sure to read the [Important Notes](README.md#%EF%B8%8F-important-notes) section in the [README](README.md) of this repository.

1. Add `CondorSetupV310.exe` as a Non-Steam game.

1. Rename the **CondorSetupV310.exe** Steam shortcut to **Condor 3**.

1. Select **Proton 11.0** as compatibility tool.

1. Launch the **Condor** Steam shortcut and immediately exit the installer so that the Proton prefix gets created.

1. Obtain the `APP_ID` of the Proton prefix:

    ```sh
    export APP_ID=$(flatpak run com.github.Matoking.protontricks -l \
        | grep "^Non-Steam shortcut: Condor 3 ([0-9]\+)$" \
        | sed -E 's/.*\(([0-9]+)\).*/\1/' \
        | head -n1)
    echo "APP ID: $APP_ID"
    ```

1. Use a Windows VM to botain an installation of Condor 3 with `CondorSetupV310.exe` and copy the `Condor3` from the VM into `/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c`

> [!IMPORTANT]
> In the following step the placeholders denoted by `<...>` must be replaced accordingly to the following table:
>
> | Placeholder | Description                                     |
> |-------------|-------------------------------------------------|
> | `<USER>`    | Your username                                   |
> | `<APP_ID>`  | The Proton prefix **APP ID** obtained in step 6 |

1. Update the **Condor 3** Steam shortcut as follows:

    **TARGET**:

    ```text
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/Condor3/Condor.exe"
    ```

    **START IN**:

    ```text
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/Condor3"
    ```

    **LAUNCH OPTIONS**:

    ```sh
    "$("$STEAM_RUNTIME"/scripts/switch-runtime.sh --runtime='' -- flatpak info -l de.bwravencl.ControllerBuddy)/files/share/proton-wrapper.sh" Condor_3 %command%
    ```

1. If you are using a controller that requires Steam Input, select the **Gamepad With Camera Controls** layout for **Condor 3** to ensure the controller will be detected by ControllerBuddy.  
In case of the Steam Deck, apply the special ControllerBuddy layout instead as described in the [Steam Deck Specifics](README.md#%EF%B8%8F-touchpad-configuration) section below.

1. Launch **Condor 3**, ignore the warning about the configuration script failure, create a pilot, and immediately exit.

1. Launch **Condor 3** a second time, this time the configuration script should succeed.

## 💡 Additional Hints

### License Activation

Both Condor 3 itself and additional planes need to be reactivated after every launch.
While the corresponding registry keys get created, Condor 3 seems to be unable to retrieve them when it is launched again.

### Joining Mutiplayer Servers

The Condoe 3 MIME type handler is not working correctly when Condor 3 is installed via Proton.
To join multiplayer servers, a few manual steps are required:

1. Open the [Server List](https://www.condorsoaring.com/serverlist/?wdt_search=cndr3) in your web browser and right-click the "Join" button of the server you want to join, then select "Copy Link Address".

> [!IMPORTANT]
> In the following step the placeholders denoted by `<...>` must be replaced accordingly to the following table:
>
> | Placeholder | Description                                                |
> |-------------|------------------------------------------------------------|
> | `<URL>`     | The URL copied from the "Join" button in the previous step |

1. Use the following command to decode the IP address and port from the URL and copy it to the clipboard:

    ```sh
    echo <URL> | tr ABCDFHJLMRSZ 03162897.5:4
    ```

1. Launch Condor 3, open the multiplayer menu, and paste the decoded IP address and port into the "Direct Connect" field to join the server.
