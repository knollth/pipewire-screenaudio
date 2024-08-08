# <img src="./extension/assets/icons/icon.svg" width="22" alt="Logo"> Pipewire Screenaudio

Extension to passthrough Pipewire audio to WebRTC Screenshare

Based on [virtual-mic](https://github.com/Curve/rohrkabel/tree/master/examples/virtual-mic) and [Screenshare-with-audio-on-Discord-with-Linux](https://github.com/edisionnano/Screenshare-with-audio-on-Discord-with-Linux)

This fork aims to provide flatpak support for Firefox, including a CLI tool for managing both Flatpak and system-package-manager-managed installations, it relies on the current rust implementation of the native part.

Not modifying any .mozilla directories at install and only installing the necessary files, then letting users apply changes to the .mozilla directories themselves through the CLI tool (reversible through `screenaudioctl reset`) might make it easier to package the native part for different distros.
This approach also makes it easier to handle special installations of firefox like flatpak.

**This only serves as a proof of concept and for now is experimental only**

## Future Works:
- Config file for custom .mozilla directories (example for forks like librewolf or floorp)
- rewrite of the CLI tool in rust
*Support for snaps is not planned* 

## Communication

You can find us on [Matrix](https://matrix.to/#/#pipewire-screenaudio:matrix.org)

## Installation

### Packages

[![AUR](https://img.shields.io/aur/version/pipewire-screenaudio?style=for-the-badge)](https://aur.archlinux.org/packages/pipewire-screenaudio)
[![AUR](https://img.shields.io/aur/version/pipewire-screenaudio-git?style=for-the-badge)](https://aur.archlinux.org/packages/pipewire-screenaudio-git)

#### NixOS Flakes

```nix
# flake.nix

{
  inputs.pipewire-screenaudio.url = "github:IceDBorn/pipewire-screenaudio";
  # ...

  outputs = {nixpkgs, pipewire-screenaudio, ...} @ inputs: {
    nixosConfigurations.HOSTNAME = nixpkgs.lib.nixosSystem {
      specialArgs = { inherit inputs; }; # this is the important part
      modules = [
        ./configuration.nix
      ];
    };
  }
}

# configuration.nix

{inputs, pkgs, ...}: {
  environment.systemPackages = with pkgs; [
    (firefox.override { nativeMessagingHosts = [ inputs.pipewire-screenaudio.packages.${pkgs.system}.default ]; })
    # ...
  ];
}
```

### Installing from Source

#### Requirements

- cargo
- pipewire
- jq

```bash
git clone https://github.com/IceDBorn/pipewire-screenaudio.git
cd pipewire-screenaudio
bash install.sh
```
Building from source installs everything locally in `$HOME/.local`

## Usage

- #### Via the extension

  - Install the [extension](https://addons.mozilla.org/firefox/addon/pipewire-screenaudio)
  - Optional: Grant extension with access permissions to all sites
  - Join a WebRTC call, click the extension icon, select an audio node and share
  - Stream, your transmission should contain both audio and video


- #### Via the CLI

  - **Description:** this fork provides `screenaudioctl`, an interactive CLI tool written in used for managing the native part
  - **Usage:**
    ```bash
    screenaudioctl --help
    user@fedora:~$ screenaudioctl
    Usage: script.sh [action] [options]
    Actions:
      apply                     Apply to mozilla folder
      list                      List destination mozilla folders
      reset                     Delete .mozilla folders, reset changes to firefox flatpak
    Options:
      -h, --help                Show help
      --installed               for listing installed .mozilla folders
      -v, --verbose             Enable verbose mode
    ```
  - **list .mozilla directories and apply to selected destination:**
    ```
    $ screenaudioctl list

    Found the following .mozilla directories:

    1. user:
      /home/user/.mozilla
    2. user-flatpak:
     /home/user/.var/app/org.mozilla.firefox/.mozilla
    ```
  - listing all .mozilla directories that have been modified by `screenaudioctl`
    ```bash
    
    $ screenaudioctl list --installed

    Found the following .mozilla directories:

    1. user:
      /home/user/.mozilla
    2. user-flatpak:
      /home/user/.var/app/org.mozilla.firefox/.mozilla
    ```
  - setting up the native part for selected .mozilla folder
    ```bash
    $ screenaudioctl apply

    Found the following .mozilla directories:

      1 . user:
        /home/user/.mozilla
      2. user-flatpak:
        /home/user/.var/app/org.mozilla.firefox/.mozilla

      Select where to apply changes to ([1-2] a=all, 0=abort): 2
    ```
    **For Flatpak:** Running apply on flatpak installs gives the flatpak filesystem permissions for `xdg-run/pipewire-0`, this is required for the native part to work. `screenaudioctl` also copies the `connector-rs` binary into .mozilla/native-message/hosts in addition to firefox.json

  - **list destinations where native part is set up and resetting selected .mozilla directories:**
    ```bash
    $ screenaudioctl reset

    Found the following .mozilla directories:

    1. user:
      /home/user/.mozilla
    2. user-flatpak:
       /home/user/.var/app/org.mozilla.firefox/.mozilla

    Select where to apply changes to ([1-2] a=all, 0=abort): 

    ```
    **For Flatpak:** For Firefox Flatpaks, this will reset the permissions to default as defined in the [**manifest**](https://hg.mozilla.org/mozilla-central/file/tip/taskcluster/docker/firefox-flatpak/runme.sh). 

## Known Problems

- You can't stream firefox WebRTC calls at all while using `All Desktop Audio`, they are excluded by default

### resistFingerprinting

- privacy.resistFingerprinting (enabled by default in LibreWolf, arkenfox user.js, etc.) breaks the extension. Either disable the preference or add any domains you wish to use Pipewire Screenaudio with to `privacy.resistFingerprinting.exemptedDomains` in `about:config`

### Audio pitching

- This bug exclusively impacts Firefox versions predating 120

## Planned Features

- Multiple nodes selection
- More customization options (node matching, watcher behavior etc.)
- Chromium support
