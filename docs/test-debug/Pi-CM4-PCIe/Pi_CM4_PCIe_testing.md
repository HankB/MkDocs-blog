# Pi CM4 PCIe testing

## Purpose

Test Debian Bookworm on a Pi CM4 with (and booting from) an NVME adapter and two SATA adapters in the "Compute Module 4 IO Board"

## Summary as of 2023-03-28

### NVME with SSD

System boots and works as expected.

### ASMedia PCIe/SATA adapter

Boot from SD card works 3-4 times out of 5. Other times halts with a blank screen following the "rainbow screen." When a SATA SSD is connected (when booting from SD) it appears to work.

Boot from SSD was unsuccessful.

### Marvel PCIe/SATA adapter

Unable to boot from SD card when this adapter is in the PCIe slot. The "rainbow screen" appears and the screen goes blank with no further (on screen) behavior. With no SD card in the slot, the firmware cycles through boot options until it gets to PXE boot and then repeats the cycle.

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

## TODO

* Report more information on the PCIe/SATA adapters.
* Report `dmesg` output for various combinations (where boot was successful.)
* Perform serial debug for "didn't boot" scenarios.
* Search for tweaks to boot CM4 from PCIe/SATA.
