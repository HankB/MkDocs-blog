# dphacks CM$ Ether Board

Expansion board for the Raspberry Pi Compute Module 4 (CM4) that supports Ethernet, USB and PCIe/NVME SSD.

This board has kindly been provided by `@makerbymistake` AKA `@rpilocator` at <dphacks.com> (and IRL André) for testing. 

Board details.

* USB-C to power the board and CM4
* Micro-USB for data with jumper to select host/device mode.  In host mode, the board provides 5V to the USB device plugged in (as determined by the USB standard). Device mode is used to flash the eMMC module. (My CM4 is Lite => no eMMC so I cannot test this.)
* Both the power and data USB ports have basic protection against ESD.
* Shorting GND and Global_EN pins (with tweezers, etc) causes the CM4 to boot up when turned off or reboot if already running.
*  M.2 M Key slot is powered by 3V (standard for SSDs). An M2 screw is proviced that fits into the post. Supported SSD sice is 2230.

## 2022-10-31 Plans

Happy Halloween!

I have a CM4 Lite with WiFi and 8GB RAM. I have been testing this with the official R-Pi expansion board (Pi-EB) including booting from NVME. For my Raspberry Pi 4B devices I prefer to use pure Debian but it will not boot the CM4 if anything is connected to the PCIe slot so it is a non-starter. I have an ARM64 R-Pi OS image installed on a 256GB Kioxia (PCIe X1) NVME SSD that I plan to test with.

The networking and NVME performance are the things I am most curious about. I may explore other areas such as USB boot and perhaps the connections for shutting down and booting the CM4.

## Baseline measurements

I plan to use `iperf3` and some common disk benchmarks to evaluate performance. Before swapping the CM4/heat sink and SSD to the CM4 Ether Board (CM4-EB) I'll complete the benchmarks on the Pi-EB. At this point I generally collect the time to complete the disk benchmarks as a first approximation of performance and will look at the detailed results if anything interesting comes up.

Before proceeding I upgraded all available packages and this provides the kernel

```text
hbarta@cm4nvme:~ $ uname -a
Linux cm4nvme 5.15.74-v8+ #1595 SMP PREEMPT Wed Oct 26 11:07:24 BST 2022 aarch64 GNU/Linux
hbarta@cm4nvme:~ $ 
```

I have Gnome installed and will disable that for this testing since the CM4-EB has no video output.

Before swapping boards the host name was changed to avoid SSH login issues (and my confusion.)

```text
sudo hostnamectl set-hostname cm4eb
```

## CM4-EB first boot

Uneventful. It came up, connected to my LAN via Ethernet and I was anle to SSH in. Results of `ip addr` indicates that WiFi is working as well (though that resides entirely on the CM4.) At present the D2 (green) LED is flickering at about 1/s and the D3 (red) LED is on continuous. On to the benchmarks!

Quick reactions - The Ethernet and NVME performance seem to match that on the official IO Board and this is both desired and expected.
