# Installing Sway on Arch Linux with NVIDIA GPUs

## Warnings

Sway does not officially support NVIDIA drivers (proprietary / open-source) except for Nouveau.

Incorrectly playing with Sway can easily brick your system (and it's not fun to deal with).

This is not an official guide by any means, and you follow it at your own risk.

Xorg / XWayland does not play very nice with NVIDIA drivers.

Sway on NVIDIA hardware is NOT stable.

## Specs

- NVIDIA RTX 3080Ti (Founders Edition)

- NVIDIA Drivers: 525.60.11

## References

- [Arch Wiki](https://wiki.archlinux.org/title/sway)

- [Hyprland](https://wiki.hyprland.org/hyprland-wiki/pages/Nvidia)

- [NVIDIA Forums](https://forums.developer.nvidia.com/t/nvidia-495-on-sway-tutorial-questions-arch-based-distros/192212)

## Getting started

### Downloading the Arch Linux ISO

Download the Arch Linux .ISO file from [https://archlinux.org/download](https://archlinux.org/download).

#### [Optional] Verify the file integrity

##### Windows

On Windows, navigate to the directory of the Arch Linux .ISO file in PowerShell. Then run `Get-FileHash archlinux.iso -Algorithm SHA256` and compare the results to sha256sums.txt.

##### Linux

On Linux, you can run `sha256sum archlinux.iso` and compare the results to sha256sums.txt.

#### Creating a bootable USB flashdrive

##### Windows

On Windows, it is recommended that you use the [Rufus](https://rufus.ie) tool to create a bootable USB for the Arch Linux installer.

There are alternative methods, but using Rufus is the easiest.

##### Linux

On Linux, it is recommended that you use [Balena Etcher](https://www.balena.io/etcher) to create a bootable USB for the Arch Linux installer.

To do so, you may have to run `chmod +x BalenaEtcher.AppImage` before being able to run the app.

There are alternative methods, but using Balena Etcher is the easiest.

### Booting the Arch Linux installer

Regardless of your Operating System, running the Arch Linux installer from the now bootable USB flashdrive should be relatively simple.

Depending on your system's BIOS, you may have to press ESC, F1, F2, F8, F12, DEL, etc. to go to the Boot Menu.

You may also not have an available Boot Menu and therefore have to specify Boot Priority in the BIOS settings. You should set the bootable USB flashdrive to the highest Boot Priority if applicable.

Once the initial dialog screen pops up, you can either press ENTER on the first option to continue straight into the Shell, or you can wait and let it auto-select the first option for you (given that you didn't interact with the system within the given delay time).

### The chroot environment

Arch Linux does not have a very straightforward installation method like other popular distributions like Ubuntu or Fedora.

You will be brought into the chroot environment. Here you can either follow the [Arch Wiki](https://wiki.archlinux.org/title/Installation_guide) or continue following the steps to come in this section.

If you have decided to follow this guide for installing Arch Linux, you will now need to set up your wireless connection. The Arch Linux installer comes preinstalled with the IWD networking tool.

Before you mess with the tool, try running `ping 8.8.8.8` (Google) or `ping 1.1.1.1` (CloudFlare) to ensure you do not have a valid internet connection. If you have a valid internet connection or you are directly connected (Ethernet), please **DO NOT** continue following these steps to set up WiFi.

#### Acquiring a WiFi connection

Run `iwctl`. This will put you into a new interactive Shell environment. Here you need to figure out what device your wireless card is. The most common identifier is "wlan0" however this may not always be applicable.

In this interactive Shell environment, run `device list`. If you see your wireless card there, take note of what its identifier is. For this guide, please acknowledge it will be assumed to be "wlan0."

If your wireless card is not listed as "Powered on," you will need to take note of its adapter identifier. Please acknowledge that it will be assumed to be "phy0" you will need to take note of its adapter identifier. Please acknowledge that it will be assumed to be "phy0." Run `adapter phy0 set-property Powered on` in the interactive Shell environment. Then run `device wlan0 set-property Powered on`. 

Afterward, a station should be already assigned to your wireless card. You can specify the station by the wireless card identifier. Run `station wlan0 scan` in the interactive Shell environment to scan for networks. This should populate the known list of networks. To view this list, run `known-networks list`. When you're finally ready to connect to one, run `station wlan0 connect SSID` replacing "SSID" with the name of your WiFi network. Verify you are connected with `device wlan0 show`.

You can exit the interactive Shell environment with `exit`.

(End of WiFi configuration.)

#### Starting DHCP services

If you're connected to WiFi or Ethernet and have verified the connection, but otherwise cannot connect to the internet, your DHCP may be to blame. All you should have to do is run `dhcpcd` in your chroot Shell, and this should start the DHCP service. You should then have a fully working internet connection.

This does not apply to every system. My desktop had no need for running `dhcpcd` whereas my laptop did, and they both use WiFi.

#### Pacman keyring initialization

You must initialize the Pacman keyring before continuing onto installing Arch Linux. It's really simple, just run `pacman-key --init` in your chroot Shell environment.

### Installing Arch Linux and Sway

Now's the fun part!

You should be able to run `archinstall`. It is not the most stable software, but you should be able to get it to run after a few tries. If not, try repopulating your Pacman keyring, rebooting your installer, or trying a different Arch Linux ISO.

It's a pretty straightforward installer, I chose the following configuration:

- Language: English

- Layout: US

- Mirror: United States

- Locale: en\_US, UTF-8

- Partition: 1 Drive (BTRFS / Subvolumes / Compression) Erase-All Defaults

- Boot Loader: Systemd Boot

- Swap: True

- Profile: Desktop - Sway - NVIDIA Open Source Modules (Turing+)

- Audio: PipeWire

- Kernel: Linux

- Network: Copy from ISO

- Optional: Multilib

Select Install and when it's done, pull out the USB flashdrive and reboot your system. Unless you are dual-booting, your system should automatically recognize the bootable system.

If it does not, you can always try using GRUB2 instead of Systemd Boot.

If you are dual-booting, be wary of bootable conflicts. When I tried installing Arch Linux, it automatically prioritized the GRUB bootable and ignored the Systemd Boot bootable drive entirely (it wasn't listed in the BIOS). However, if you are dual-booting, it might just make more sense to just add the entry to your GRUB settings anyhow.

#### Downloading the right drivers

The NVIDIA Open Source Modules option for newer GPUs will not install the best option for Sway. This will install the `nvidia-open` package. You will need the `nvidia-open-dkms` package, as well as `linux-headers`.

To do this, it's pretty simple: run `sudo pacman -S nvidia-open-dkms linux-headers` and select enter "Y" to verify any overrides.

Please make sure the driver versions are at least over 500. My system tested with version 525.

You may want to go ahead and reboot your system. Ensure that when you run `nvidia-smi` it doesn't return with no devices found, it should display information about your primary GPU as well as the NVIDIA drivers.

Now you will need to update those drivers. (This may warrant another reboot LOL.) Run `modprobe nvidia NVreg_OpenRmEnableUnsupportedGpus=1` and `modprobe nvidia_drm modeset=1` (alt. try `modprobe nvidia nvidia_drm.modeset=1` -- probably incorrect).

While I haven't tested if it makes a difference, you should run `mkinitcpio` afterward just to be safe. You can verify what modules are getting loaded with `mkinitcpio --auto-mods`.

#### Setting up Seatd and Polkit

First, just make sure you have Polkit installed. This is technically optional, but I'd suggest you do it anyway. Run `sudo pacman -S polkit`. This may help with compatibility.

Next, take note of your username. You will need to add your user profile to the "seat" group. Run `sudo usermod -a -G seat username` replacing "username" with your username.

Finally, ensure that Seatd will run on system start. Run `sudo systemctl enable seatd`.

#### Setting up your .bash\_profile

You need to make sure some environment variables are assigned for NVIDIA compatibility before you try to run Sway.

Run `vim ~/.bash_profile` to edit your Bash profile.

Paste the following text somewhere appropriate in the file.

```bash
export LIBVA_DRIVER_NAME=nvidia
export XDG_SESSION_TYPE=wayland
export GBM_BACKEND=nvidia-drm
export __GLX_VENDOR_LIBRARY_NAME=nvidia
export WLR_NO_HARDWARE_CURSORS=1
export MOZ_ENABLE_WAYLAND=1 # (REQUIRED: Firefox will break otherwise.)
#export WLR_RENDERER=vulkan (WARNING: This breaks for me.)
```

If you don't know how to use Vim, you could also use `nano` instead.

Pressing I in interactive (default) mode will put you into insert mode.

Pressing ESC in insert mode will put you in interactive mode.

Typing `:wq` in interactive mode and then pressing ENTER will quit and save the file when you're ready.

### Sway Configuration

Make sure you at least have a configuration file available for Sway. You do not need to specify XWayland if you want support for Xorg applications.

Run the following commands to create the configuration file: `mkdir -p ~/.config/sway && cp -r /etc/sway/config ~/.config/sway/config`.

### Display

If you're like me and have a 1440p240Hz display, you might need to configure your display settings. Firstly, we need to figure out which display is which. I only have one, but you may have multiple. Run `swaymsg -t get_outputs` and take note of the information. Specifically, which display resolution and refresh rate is associated with which monitor identifier.

`vim ~/.config/sway/config` and somewhere put `output display mode resolution@xHz` replacing "display" with the respective display identifier, "resolution" with the respective screeb resolution, and "x" (before "Hz") with the screen hertz.

For me, this looks like: `output DP-1 mode 2560x1440@239.761Hz`. 

If (and only if) you experience flickering, you need to run `modprobe nvidia NVreg_RegistryDwords="PowerMizerEnable=0x1; PerfLevelSrc=0x2222; PowerMizerLevel=0x3; PowerMizerDefault=0x3; PowerMizerDefaultAC=0x3"`. This will force your GPU into performance mode, entailing more power usage from your GPU.

Another possible fix is to go through all the Sway source code, replacing `glFlush();` with `glFinish();`, but that entails building Sway from source.

A final possible fix is to use AUR packages, such as `sway-nvidia` and `wlroots-nvidia`. This may actually allow compatibility with fully proprietary NVIDIA drivers. So instead of having `nvidia-open` and `nvidia-open-dkms`, you'd instead have `nvidia` and possibly `nvidia-dkms`. In the end, it just depends on your system. If you decide to try the proprietary drivers, DO NOT do `sudo pacman -R nvidia-open nvidia-open-dkms` but instead do `sudo pacman -S nvidia nvidia-dkms` as it will automatically handle all the dependencies for you and avoid errors. The following option 1 will be for installing `sway-nvidia` from source, option 2 for using an AUR helper to install `sway-nvidia`, option 3 will be for installing `wlroots-nvidia` from source, and (**RECOMMENDED FOR BEST RESULTS**) option 4 for using an AUR helper to install `wlroots-nvidia`. 

1) Run `sudo pacman -S git`. Then run `git clone https://aur.archlinux.org/sway-nvidia.git`. Enter the directory with `cd sway-nvidia` and then run `makepkg -si`. (This will NOT automatically receive updates.)

2) Run `sudo pacman -S git`. A list of helpers are available on the [Arch Wiki](https://wiki.archlinux.org/title/AUR_helpers), but this example will show `paru`. Run `git clone https://aur.archlinux.org/paru.git`. Enter the directory with `cd paru` and then run `makepkg -si`. You will then use `paru -S nvidia-sway` to install the patch. Then in your "~/.bash_profile" you need to replace `exec sway --unsupported-gpu` to `exec sway-nvidia`.

3) Run `sudo pacman -S git`. Then run `git clone https://aur.archlinux.org/wlroots-nvidia.git`. Enter the directory with `cd wlroots-nvidia` and then run `makepkg -si`. (This will NOT automatically receive updates.)

4) Run `sudo pacman -S git`. A list of helpers are available on the [Arch Wiki](https://wiki.archlinux.org/title/AUR_helpers), but this example will show `paru`. Run `git clone https://aur.archlinux.org/paru.git`. Enter the directory with `cd paru` and then run `makepkg -si`. You will then use `paru -S wlroots-nvidia` to install the patched version of wlroots. Do not try to remove the originally installed wlroots, let `paru` do its thing.

The best working solution for me was using `wlroots-nvidia` with the open source dkms NVIDIA modules. Unfortunately `sway-nvidia` caused frequent "out of memory" errors that locked up my kernel and forced a reboot, even with using `timeout`.

To update applications installed with `paru`, simply run `paru`.

There are also applications you can install that have the ability to change the screen resolution at runtime.

### Audio

You should not need to do any specific configuring for proper audio. The Arch Linux installer (assuming you followed my guide) would have already configured it and installed `pavucontrol`. If you want to change your output/input device and/or the volume, simply run `pavucontrol` and it will pop up a GUI application for configuration.

### Trying out Sway

Now, if all's ready to go, you should be able to try out Sway.

Just a heads up: I'd recommend rebooting before trying this. It might save you some time.

Whenever you're ready, run `timeout 1m sway --unsupported-gpu -d`. This will run Sway for 1 minute. If something goes wrong, worst case scenario, it'll exit after 60 seconds and will provide you error logs.

If it does work perfectly fine, you may be ready to go ahead and put the sway script into your Bash profile.

### Auto-start Sway

Run `vim ~/.bash_profile` and simply add this line below the text you copied and pasted: `exec sway --unsupported-gpu`.

Do not run Sway in `sudo`. It should only have user mode privileges.

### XWayland

XWayland is enabled by default. Do not try to enable it in the Sway configuration file. It's bad news bears.

### Extra packages I installed

I'm not sure if these will help you out or not in your endeavors, but I also ran the following command: `sudo pacman -S lib32-mesa lib32-keyutils lib32-nvidia-utils lib32-nvidia-cg-toolkit opencl-nvidia vulkan-headers`.

### Exiting Sway

If your Sway is broken and you see a blank screen you cannot interact with, there's nothing you can do but reboot. However, you can exit from your normal Sway instance with SUPER+SHIFT+E. SUPER is typically either the Windows key or Alt
.
