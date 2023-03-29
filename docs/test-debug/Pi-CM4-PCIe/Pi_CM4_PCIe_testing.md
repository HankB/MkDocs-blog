# Pi CM4 PCIe testing

## Purpose

Test Debian Bookworm on a Pi CM4 with (and booting from) an NVME adapter and two SATA adapters in the "Compute Module 4 IO Board"

## Results as of 2023-03-28

### NVME with SSD

System boots from NVME and works as expected. The system will *not* boot from SD card when the PCIe/NVME adapter is installed.

### ASMedia PCIe/SATA adapter

Boot from SD card works 3-4 times out of 5. Other times halts with a blank screen following the "rainbow screen." When a SATA SSD is connected (when  booting from SD) it appears to work.

Boot from SSD was unsuccessful. The "rainbow screen" did not appear but the subsequent (BIOS) screen looping through possible boot modes (`NETWORK` and `BCM-USB-MSD`) At some point something was printed to the serial port but it looked like a single character at the wrong baud rate.

Many times the system was power cycled with both SD card in place and SSD connected and the result was a blank screen and no serial output. Once the system came up and appeared to boot from the SD card and find root on the SSD. After sevaral tries I duplicated that and captured the [serial output.](./minicom-SD_boot-SSD_root.cap.txt
)

```text
root@cm4iob:~# lspci -v
00:00.0 PCI bridge: Broadcom Inc. and subsidiaries BCM2711 PCIe Bridge (rev 20) (prog-if 00 [Normal decode])
        Flags: bus master, fast devsel, latency 0, IRQ 23
        Bus: primary=00, secondary=01, subordinate=01, sec-latency=0
        Memory behind bridge: 00000000-000fffff [size=1M] [32-bit]
        Prefetchable memory behind bridge: [disabled] [64-bit]
        Capabilities: [48] Power Management version 3
        Capabilities: [ac] Express Root Port (Slot-), MSI 00
        Capabilities: [100] Advanced Error Reporting
        Capabilities: [180] Vendor Specific Information: ID=0000 Rev=0 Len=028 <?>
        Capabilities: [240] L1 PM Substates
        Kernel driver in use: pcieport

01:00.0 SATA controller: ASMedia Technology Inc. Device 1064 (rev 02) (prog-if 01 [AHCI 1.0])
        Subsystem: ZyDAS Technology Corp. Device 2116
        Flags: bus master, fast devsel, latency 0, IRQ 38
        Memory at 600080000 (32-bit, non-prefetchable) [size=8K]
        Memory at 600082000 (32-bit, non-prefetchable) [size=8K]
        Expansion ROM at 600000000 [virtual] [disabled] [size=512K]
        Capabilities: [40] Power Management version 3
        Capabilities: [50] MSI: Enable+ Count=1/1 Maskable- 64bit+
        Capabilities: [80] Express Endpoint, MSI 00
        Capabilities: [100] Advanced Error Reporting
        Capabilities: [130] Secondary PCI Express
        Kernel driver in use: ahci
        Kernel modules: ahci

root@cm4iob:~# 
```

### Marvel PCIe/SATA adapter

Unable to boot from SD card when this adapter is in the PCIe slot. The "rainbow screen" appears and the screen goes blank with no further (on screen) behavior. With no SD card in the slot, the firmware cycles through boot options until it gets to PXE boot and then repeats the cycle.

With the SD card removed and a bootable SSD connected to the adapter, the "rainbow screen" does not appear and there is no output to the serial port.

Card dettails (from R-Pi OS)

```text
root@piserver:~# lspci -v
00:00.0 PCI bridge: Broadcom Inc. and subsidiaries BCM2711 PCIe Bridge (rev 20) (prog-if 00 [Normal decode])
        Device tree node: /sys/firmware/devicetree/base/scb/pcie@7d500000/pci@0,0
        Flags: bus master, fast devsel, latency 0
        Bus: primary=00, secondary=01, subordinate=01, sec-latency=0
        I/O behind bridge: 00000000-00000fff [size=4K]
        Memory behind bridge: c0000000-c00fffff [size=1M]
        Prefetchable memory behind bridge: [disabled]
        Capabilities: [48] Power Management version 3
        Capabilities: [ac] Express Root Port (Slot-), MSI 00
        Capabilities: [100] Advanced Error Reporting
        Capabilities: [180] Vendor Specific Information: ID=0000 Rev=0 Len=028 <?>
        Capabilities: [240] L1 PM Substates

01:00.0 SATA controller: Marvell Technology Group Ltd. 88SE9215 PCIe 2.0 x1 4-port SATA 6 Gb/s Controller (rev 11) (prog-if 01 [AHCI 1.0])
        Subsystem: Marvell Technology Group Ltd. 88SE9215 PCIe 2.0 x1 4-port SATA 6 Gb/s Controller
        Device tree node: /sys/firmware/devicetree/base/scb/pcie@7d500000/pci@0,0/usb@0,0
        Flags: bus master, fast devsel, latency 0, IRQ 67
        I/O ports at 0000
        I/O ports at 0000
        I/O ports at 0000
        I/O ports at 0000
        I/O ports at 0000
        Memory at 600010000 (32-bit, non-prefetchable) [size=2K]
        Expansion ROM at 600000000 [disabled] [size=64K]
        Capabilities: [40] Power Management version 3
        Capabilities: [50] MSI: Enable+ Count=1/1 Maskable- 64bit-
        Capabilities: [70] Express Legacy Endpoint, MSI 00
        Capabilities: [e0] SATA HBA v0.0
        Capabilities: [100] Advanced Error Reporting
        Kernel driver in use: ahci
        Kernel modules: ahci

root@piserver:~#
```

Capture of [serial output](./minicom-SD_Raspbian-Marvell.cap.txt) during R-Pi OS boot.

## NVME details

```text
root@debnvme:~# lspci -v
00:00.0 PCI bridge: Broadcom Inc. and subsidiaries BCM2711 PCIe Bridge (rev 20) (prog-if 00 [Normal decode])
        Flags: bus master, fast devsel, latency 0, IRQ 23
        Bus: primary=00, secondary=01, subordinate=01, sec-latency=0
        Memory behind bridge: 00000000-000fffff [size=1M] [32-bit]
        Prefetchable memory behind bridge: [disabled] [64-bit]
        Capabilities: [48] Power Management version 3
        Capabilities: [ac] Express Root Port (Slot-), MSI 00
        Capabilities: [100] Advanced Error Reporting
        Capabilities: [180] Vendor Specific Information: ID=0000 Rev=0 Len=028 <?>
        Capabilities: [240] L1 PM Substates
        Kernel driver in use: pcieport

01:00.0 Non-Volatile memory controller: Toshiba Corporation XG5 NVMe SSD Controller (prog-if 02 [NVM Express])
        Subsystem: Toshiba Corporation XG5 NVMe SSD Controller
        Flags: bus master, fast devsel, latency 0, IRQ 38, NUMA node 0
        Memory at 600000000 (64-bit, non-prefetchable) [size=16K]
        Capabilities: [40] Express Endpoint, MSI 00
        Capabilities: [80] Power Management version 3
        Capabilities: [90] MSI: Enable+ Count=1/32 Maskable+ 64bit+
        Capabilities: [b0] MSI-X: Enable- Count=32 Masked-
        Capabilities: [100] Advanced Error Reporting
        Capabilities: [260] Latency Tolerance Reporting
        Capabilities: [300] Secondary PCI Express
        Capabilities: [400] L1 PM Substates
        Kernel driver in use: nvme
        Kernel modules: nvme

root@debnvme:~# 
```

`smartctl` identifies the drive as

```text
Model Number:                       KXG50ZNV512G NVMe TOSHIBA 512GB
Serial Number:                      xxxxxxxxxxxx
Firmware Version:                   AADA4106
PCI Vendor/Subsystem ID:            0x1179
IEEE OUI Identifier:                0x00080d
```

## 2023-03-28 Serial debug

Serial connection worked with the boot image but produced no output for the Marvell adapter. There was no change with the changes listed below despite power cycling it repeatedly.

Comments per `kibi` on `#debian-raspberrypi`

```text
<kibi> if the kernel doesn't boot, and if it doesn't very early, nearly no chances of getting logs stored onto the internal FS (or external SD card), since / is unlikely to be mounted at this stage
<kibi> anyway:
<kibi> early_con → /etc/default/raspi-extra-cmdline
<kibi> printf "enable_jtag_gpio=1\nforce_turbo=1\n" >> /etc/default/raspi-firmware-custom
<kibi> printf CONSOLES=\"ttyS1,115200\" >> /etc/default/raspi-firmware
```

Confirmation

```text
root@cm4iob:~# cat /etc/default/raspi-extra-cmdline
early_con
root@cm4iob:~# cat /etc/default/raspi-firmware-custom
enable_jtag_gpio=1
force_turbo=1
root@cm4iob:~# tail /etc/default/raspi-firmware
#KERNEL_ARCH="arm64"

# Create a file "/etc/default/raspi-firmware-custom" to add custom parameter
# to startup the kernel. Maybe not all options are supported.
# (see https://www.raspberrypi.com/documentation/computers/config_txt.html)
#
# To pass extra arbitrary parameters to the kernel at boot, you can specify
# them in "/etc/default/raspi-extra-cmdline". Keep in mind they should be
# all in a single line, no comments!
CONSOLES="ttyS1,115200"root@cm4iob:~# 
root@cm4iob:~# 
```

## 2023-03-29 focus on SD boot failure when PCIe card is present

### Marvell PCIe/SATA adapter

Successful boot [dmesg](./dmesg-SD-Debian-ASMedia_successful.txt) and [serial](./minicom-SD-Debian-ASMedia_success.cap.txt)

## TODO

* ~~Report more information on the PCIe/SATA adapters.~~
* ~~Report `dmesg` output for various combinations (where boot was successful.)~~
* ~~Perform serial debug for "didn't boot" scenarios.~~
* ~~Search for tweaks to boot CM4 from PCIe/SATA.~~
