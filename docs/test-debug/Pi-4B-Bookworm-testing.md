# Pi 4B Bookworm testing

Determine if some previous bugs have been fixed in more recent kernels. Bugs fell into two areas:

* vc4 module - this included a use-after-free as well as other issues reported in `dmesg`.
* SD card timeout - Timeout message repeated every 10s. This had manifested previously and a workaround was to insert an SD card into the slot and the messages stopped. This time that does not work. Further characteristics of the present situation:
  * doesn't boot
  * Boots from attached USB SSD skipping the SD card.
  * No /dev/mmc* after boot accompanied by no WiFi.
  * /dev/mmc* entries, can mount and read card and after several more timeout messages, they stop.
  * When the timeout messages are being reported, the system cannot complete shutdown to reboot. It is necessary to remove power from the system.
  * Normal boot, no mmc timeouts and apparently normal operation.

In other words, there is a great variety of behavior with kernels with Bookworm. The good news is that later kernels do not exhibit the problems with the vc4 module so the focus on these notes is to better characterize the SD timeout issue now reported at <https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1019700>.

## Procedure

1. Start with a fresh install of `20220808_raspi_4_bookworm.img.xz` On an SD card.
1. Boot, configure a user to facilitate access via SSH.
1. Examine `dmesg` output to determine
   1. SD timeouts
   1. `/dev/mmc*` emtries
   1. indication of WiFi functionality (via `ip addr`)
1. Reboot mutiple times to determine pf the SD timeout problem exists.
1. Update/upgrade to get new kernel and repeat checks above including multiple reboots.
1. find and install other kernels and repeat checks/reboots.

## Results

### Bullseye

```text
Linux cm4pi 5.15.61-v8+ #1579 SMP PREEMPT Fri Aug 26 11:16:44 BST 2022 aarch64 GNU/Linux
```

* No SD timeouts (booted from SD)
* WiFi working
* `/dev/mmc*` entries present

### Bookworm

Installed kernel

```text
Linux muon 5.18.0-3-arm64 #1 SMP Debian 5.18.14-1 (2022-07-23) aarch64 GNU/Linux
```

* Came up on first install w/out difficulty.
* Rebooted and power cycled, 3 times each. No SD problem.
* update/upgrade and reboot.

```text
Linux muon 5.19.0-1-arm64 #1 SMP Debian 5.19.6-1 (2022-09-01) aarch64 GNU/Linux
```

* On first boot mmc timeout messages are appearing. Delaying completoin of boot. Remounting SD card. Cannot remote in. After waiting a couple minutes, tried C-A-D and no response. power cycled. 
* Second boot OK.