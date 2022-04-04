# Raspberry Pi

## Pi 4B

Almost good enough to be a daily driver. Even overclocked, it can't play Youtube videos full screen smoothly. The main thing that hampers it IMO us the USB attachment to an SSD. I won't run one of these off an SD card as that's just not fast enough. My dream system for the Pi 4 is a compuyte module with a carrier that supports 4 SATA ports and has enough MMC to boot/root on.

I've been preferentially running Debian on the one I use as a desktop. Raspberry Pi OS (R-Pi OS) is good enough for servers but I find the desktop too constraining. And I can't install my preferred desktop - Gnome - due to missing dependencies. I can install KDE and that's a whole 'nother discussion.

I bounce back and forth between Bullseye (stable) and Bookworm (testing.) When I get bored of no drama on Bullseye I swap in an SSD that has Bookworm. I've submitted two bug reports so far, one of which was just closed this morning (2022-04-04.)

Here are some tips for running Debian on a Pi 4B.

* The system will suspend just fine but I can't figure out how to unsuspend. The first thing I do on a new install is go into gnome settings and disable suspend. Then I tell Systemd and gnome not to suspend using the following commands. (Seems not to want to suspend with no DE installed.)

```text
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type 'nothing'
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'
```

* If you install Bookworm, hold the `raspi-firmware` package until the fixed firmware is in the Testing repo. (`apt-mark hold raspi-firmware`) Or install from experimental Or install from the tagged version `1.20220331`.
* The default setting for Ethernet is "unmanaged." It can be controlled using `ifdown eth0` / `ifup eth0` (but will be on again following the next reboot.) To let `NetworkManager` control, edit `/etc/NetworkManager/NetworkManager.conf` and change `managed=true` to `managed=false`. I'm not sure this is a complete fix for this issue as Ethernet will be back on the next boot regardless of the setting.
* Status of Bookworm seems pretty shaky right now. I get frequent loss of connection to the SSD and the only recovery is to pull power. Loss of connection to the SSD can lead to file corruption and may be the reason that my Bookworm installations seem to deteriorate with time.
