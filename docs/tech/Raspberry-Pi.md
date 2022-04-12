# Raspberry Pi

## Pi 4B

Almost good enough to be a daily driver. Even overclocked, it can't play Youtube videos full screen smoothly. The main thing that hampers it IMO us the USB attachment to an SSD. I won't run one of these off an SD card as that's just not fast enough. My dream system for the Pi 4 is a compute module with a carrier that supports 4 SATA ports and has enough MMC to boot/root on.

I've been preferentially running Debian on the one I use as a desktop. Raspberry Pi OS (R-Pi OS) is good enough for servers but I find the desktop too constraining. And I can't install my preferred desktop - Gnome - due to missing dependencies. I can install KDE and that's a whole 'nother discussion.

I bounce back and forth between Bullseye (stable) and Bookworm (testing.) When I get bored of no drama on Bullseye I swap in an SSD that has Bookworm. I've submitted two bug reports so far, one of which was just closed this morning (2022-04-04.)

Here are some tips for running Debian on a Pi 4B.

* The system will suspend just fine but I can't figure out how to unsuspend. The first thing I do on a new install is go into gnome settings and disable suspend. Then I tell Systemd and gnome not to suspend using the following commands. (Seems not to want to suspend with no DE installed.)

```text
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type 'nothing'
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'
```

* If you install Bookworm, hold the `raspi-firmware` package until the fixed firmware is in the Testing repo. (`apt-mark hold raspi-firmware`) Or install from experimental. Or install from the tagged version `1.20220331`.
* The default setting for Ethernet is "unmanaged." It can be controlled using `ifdown eth0` / `ifup eth0` (but will be on again following the next reboot.) To let `NetworkManager` control, edit `/etc/NetworkManager/NetworkManager.conf` and change `managed=true` to `managed=false`. I'm not sure this is a complete fix for this issue as Ethernet will be back on the next boot regardless of the setting.
* Status of Bookworm seems pretty shaky right now. I get frequent loss of connection to the SSD and the only recovery is to pull power. Loss of connection to the SSD can lead to file corruption and may be the reason that my Bookworm installations seem to deteriorate with time.
* Using an SSD. You will really want a powered USB hub for this. SSDs that worked fine with a Pi 3B may not work well with a 4B because USB 3 supports the faster UAS driver and in turn that causes the SSD to require more power. You can disable UAS using `usb-storage.quirks` but I prefer the faster faster speed that UAS provides. I use the Wavlink Model: WL-UH3042P1 and power the Pi off the quick charge port. It's rated for 2.4A so I plug other accessories (mouse, keyboard) into the hub along with the SSD to reduce the power demands from the Pi.
* Overclocking is relatively straight forward. On my Nullseye install I have added the following to `/boot/firmware/config.txt`

```text
arm_freq=2000
over_voltage=6
gpu_freq=750
```

And have stress tested using a variety of stressers. The CPU temperature goes up to about 82°C max. I would be concerned if I was running that kind of load at all times, but I'm not.
* CPU case. I like the FLIRC cases a lot. Big hunk of aluminum with a molded in protrusion that contacts the CPU through a small pad and does a great job of drawing heat from the processor. It's compoletely passive so it won't stop working if a fan fails because there is no fan needed.

## Pi Zero, W, 2W and others

* Overlay FS and `unattended-upgrades` - No! I use some of these essentially as embedded systems so it seemed natural to employ the Overlay File System (R/O FS, Overlay FS) to reduce SD card failure. I run some of these on really cheap SD cards. I also prefer to keep `unattended-upgrades` installed to stay current with security upgrades. Unfortunately the two don't mix very well. The Overlay FS "overlays" the R/O FS with RAM so when `apt-get update` runs, it fills RAM with disk changes and results in "no space on /".

On hosts that I have enabled Overlay RS, I have had to uninstall `unattended-upgrades`. I should look into PXE boot to see if systems that run from that can stay up to date and of course, not burn up SD cards. Unfortunately WiFi boot is problematic due to the need to establish an association before anything else can happen.
