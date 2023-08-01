# New Raspberry Pi CM4 WiFi testing

Explore WiFi on recently acquired Raspberry Pi Compute Mocule Lite with WiFi and Bluetooth.

## Motivation

1. While provisioning the new CM4 it was noted that many IoT devices were droppinig from the LAN including TP-Link Kasa lights and outlets and Raspberry Pi Zeroes. This is not unusual, particularly the TP-Link devices, but it seemed that nearly half of the devices were disconnected and that is unusual.
1. At that time I thought to turn off the radio on the new CM4 and discovered that `wlan0` did not appear in the output of `ip addr`.

These may or may not be related. The milestones I can recall before taking a step back to investigate WiFi were:

1. Install a NVME SSD with a copy of R-Pi OS (which has not been used for a while) on the Ether Board <https://hankb.github.io/MkDocs-blog/tech/dphacks-CM4_Ether_Board/>, attach the new CM4 to the board and boot. The board was connected to my LAN via Ethernet.
1. Find the device on my LAN ans SSH in. I discovered that this install was R-Pi OS and wanted to use Debian for this system so I recorded the output of `ip addr` and shut the system down. (I needed the MAC addresses for Ethernet and WiFi to remap to avoid later confusion.)
1. Install the most recent "tested Debian" build on the card including some additional setup. The image installed was `20230612_raspi_4_bookworm.img.xz` downloaded from <https://raspi.debian.net/tested-images/>
1. Completed initial configuration using the Ansible Playbooks at <https://github.com/HankB/polana-ansible>.
1. Continued configuration and noticed that many devices on my IoT LAN were disconnected from my Access Point (AP.) I power cycled the AP and that did not seem to help. I was concerned about possible RF interference from the new CM4 since it was located about 1 foot from the AP. I was not planning on using WiFi so I thought to disable that radio in case it was the cause of the WiFi issues. That was when I noticed that `ip addr` no longer displayed a device `wlan0`.

At this point I began a less than systematic exploration of the "missing `wlan0`" situation. Before I describe that, I'll list the CM4 related equipment I'm using.

* CM4 Lite with WiFi/Ethernet and 8GB RAM. I have been using this for quite a while now. I will call it the "CM4/8GB" in the following discussion.
* Official Raspberry Pi IO Board (on which the CM4/8GB is presently mounted.)
* PCIe/NVME adapter on which I can mount an NVME SSD and boot either R-Pi OS or Debian.

* CM4 Lite with WiFi/Ethernet and 2GB RAM. This is recently acquired.
* Dphacks Ether Board. I used the CM4/8GB on this for initial testing and confirmed that everythging I tested (including WiFi) worked as expected. When I received the CM4/2GB, I installed it on this board and proceeded with configuration.

* 256GB KIOXIA NVME SSD which (along with an NVME/USB adapter) I used for initial configuration and plan to use for further testing.
* 128GB SK Hynix NVME SSD on which is installed a fully updated and overclocked 64 bit R-Pi OS. This was used during the initial exploration but will not be used for the more systematic testing.

## initial exploration

I performed the following steps, not necessarily in the listed order, to tryt to isolate the issue. I did not record each step as I tested, hence the need to retest. Also I used existing installations which had been customized includinig overclocking.

1. Swap the 128GB SSD to the Ether Board/CM4/2GB and boot. Confirm that `wlan0` does not appear in `ip addr`. Confirm that there are no messages that include `wlan0` or `wifi` in `dmesg` output. Confirm that there are Bluetooth (BT) related messages in `dmesg` output. The common factor hewre is the CM4/2GB and Ether Board.
1. I believe I checked for (and found) `wlan0` with the CM4/8GB in the IO Board and running R-Pi OS from the 128GB SSD.
1. Install the 256GB SSD (Debian) on the IO Board with the CM4/8GB. The `wlan0` device was not present. The common factor here is the S/W.

At this point I saw the need to perform a more systematic investigation to try to narrow down the reason for the missing `wlan0` (and possible related WiFi issues.)

1. Test with pristine R-Pi OS image (2023-05-03-raspios-bullseye-arm64-lite.img.xz.) No joy here. On first boot it loops in keytboard configuration and I cannot find a way to bypass that. I booted R-Pi OS (old installation) from SD card and `wlan0` is not present, but wifi is disabled in `/boot/config.txt`. Commented that out, rebooted and still no `wlan0` but I could have done something erlse like blacklisting the modules. Undid that and `wlan0` is present and appears to be working (obtained IP addr using DHCP.) I wrote the same image to an SD card and was able to boot w/out any issue. And the `wlan0` device was present. The next test is to copy the SD card to the NVME SSD and boot from that.
1. Image initialized on an SD card and copied to NVME SSD booted CM4/8GB/IO Board w/out difficulty and `wlan0` was present.
1. Transferred the NVME SSD to the CM4/2GB/Ether Board. System booted, logged in and found that `wlan0` was present.
1. Performed an upgrade (still on CM4/2GB/Ether Board) and rebooted. `wlan0` still present.

In other words, on both H/W setups, `wlan0` was present.
