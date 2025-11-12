# Strike Fighters / Wings Over Vietnam / Wings Over Europe / Wings Over Israel with ControllerBuddy on Linux

### 🚧 Under Construction 🚧

1. Add `Setup.exe` as non-Steam game entry
2. Rename the entry to `Strike Fighters` / `Wings Over Vietnam` / `Wings Over Europe` / `Wings Over Israel`
3. Select `Proton 9.0-4` as compatibility tool
4. Launch the non-Steam game entry and install the game
5. Obtain the `APP_ID`:
    ```sh
    flatpak run com.github.Matoking.protontricks -l
    ```
6. Download and install the corresponding Sep 2008 update from [here](https://thirdwire.com/downloads_archive.htm)
    ```sh
    flatpak run com.github.Matoking.protontricks -c 'wine <UPDATER>.exe' <APP_ID>
    ```
7. Install DirectPlay:
   ```sh
   flatpak run com.github.Matoking.protontricks <APPID> directplay
   ```
8. Download [`disable_xbox_keychron.reg`](https://gist.github.com/bwRavencl/c8e95ac194bcce8396f6c13c7b4a8a70) and import it within `regedit`: 
   ```sh
   flatpak run com.github.Matoking.protontricks -c 'wine reg import disable_xbox_keychron.reg' <APP_ID>
   ```
9. Configure the game for the ControllerBuddy-Profile:
    ```sh
    cp ~/Projekte/ControllerBuddy/ControllerBuddy-Profiles/configs/Strike_Fighters/Default.ini ~/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/<GAME_FOLDER>/Controls/Default.ini
    ```
10. Modify the Steam entry as follows:
  - TARGET:
    ```
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/<GAME_FOLDER>/<EXE>.exe"
    ```
  - START IN:
    ```
    "/home/<USER>/.local/share/Steam/steamapps/compatdata/<APP_ID>/pfx/drive_c/<GAME_FOLDER>"
    ```
  - LAUNCH OPTIONS:
    ```sh
    ${STEAM_RUNTIME}/scripts/switch-runtime.sh --runtime='' -- flatpak run de.bwravencl.ControllerBuddy -autostart local -profile /app/share/ControllerBuddy-Profiles/Strike_Fighters.json -tray & timeout 10 bash -c 'until grep -q "ControllerBuddy Joystick" /proc/bus/input/devices ; do sleep 1 ; done' && %command% || { [ $? -eq 124 ] && zenity --error --text='ControllerBuddy not ready' ; } ; killall -q ControllerBuddy
    ```
