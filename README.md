# ControllerBuddy-Linux-Guides

## 📖 Description

This repository contains step-by-step guides for setting up various flight simulation games for use with [ControllerBuddy](https://controllerbuddy.org) on Linux.

These guides are intended to help users install and configure their flight simulators to work with ControllerBuddy and integrate seamlessly into a Steam-based gaming experience on Linux systems, including the [Steam Deck](https://www.steamdeck.com).  
This includes automatically launching and stopping ControllerBuddy alongside the respective game from within Steam.

Although the guides are designed to be as straightforward as possible, they assume a basic familiarity with Linux command-line operations and the use of [Flatpak](https://flatpak.org) applications.

## 📚 Guides

- [DCS World](DCS_World.md)
- [EF2000 V2.0](EF2000_V2.0.md)
- [Falcon 3.0](Falcon_3.0.md)
- [Falcon BMS](Falcon_BMS.md)
- [F-22: Air Dominance Fighter](F-22_ADF.md)
- [Jane's Fighters Anthology](Fighters_Anthology.md)
- [IL-2 Sturmovik: 1946](IL-2_1946.md)
- [IL-2 Sturmovik: Great Battles](IL-2_GB.md)
- [Strike Fighters](Strike_Fighters.md)

## ⚠️ Important Notes

Below are some important notes to keep in mind when following the guides in this repository. They apply to all guides, so please read them carefully before starting.

### 📋 Copying and Pasting Commands

All shell commands in the guides are meant to be copied and pasted in one go, even if they span multiple lines.

Use the copy button at the top right of each code block to copy the entire block to your clipboard, then paste it into your terminal.

### 🧩 Prerequisites

The guides assume that you have already installed the necessary prerequisites as listed in each specific guide.

It is assumed that the distribution package of Steam is used, **not** the Flatpak version.

For all other prerequisites, it is assumed that the Flatpak versions are used as linked in the respective guides. ControllerBuddy itself is not available on Flathub, but from its own Flatpak repository. Follow the instructions in the [ControllerBuddy-Flatpak](https://github.com/bwRavencl/ControllerBuddy-Flatpak) repository to set it up.

### 🪜 Following the Steps

It is crucial to follow the steps in the guides in the exact order they are presented.

Skipping steps or performing them out of order may lead to an improperly configured setup. If a step is optional, it will be explicitly marked as such.

Pay attention to the output of each command. Error messages can provide vital information regarding the success of each step.

## ⚖️ License

[CC0 1.0 Universal](LICENSE)
