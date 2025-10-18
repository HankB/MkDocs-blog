# Pi Firmware 2024-04-28

## Motivation

Notification of new firmware via IRC channel provided by `@gwolf`.

Download from <http://people.debian.org/~gwolf/raspi-firmware_1.20240424+ds-1_all.deb> Now available from <https://packages.debian.org/sid/raspi-firmware>

## 2024-04-29 CM4/Trixie install

Tested with CM4 running fully up to date Trixie and no issues noted. WiFi device still not present. The following error messages are seen in `dmesg` output.

```text
[   15.701315] brcmfmac mmc0:0001:1: firmware: failed to load brcm/brcmfmac43455-sdio.raspberrypi,4-compute-module.bin (-2)
[   15.701324] firmware_class: See https://wiki.debian.org/Firmware for information about missing firmware
[   15.701449] brcmfmac mmc0:0001:1: firmware: failed to load brcm/brcmfmac43455-sdio.raspberrypi,4-compute-module.bin (-2)
[   15.701454] brcmfmac mmc0:0001:1: Direct firmware load for brcm/brcmfmac43455-sdio.raspberrypi,4-compute-module.bin failed with error -2
[   15.705868] usbcore: registered new interface driver brcmfmac
[   15.733657] bcm2835_mmal_vchiq: module is from the staging directory, the quality is unknown, you have been warned.
[   15.736147] brcmfmac mmc0:0001:1: firmware: direct-loading firmware brcm/brcmfmac43455-sdio.bin
[   15.769863] bcm2835_v4l2: module is from the staging directory, the quality is unknown, you have been warned.
[   15.780030] Bluetooth: HCI UART driver ver 2.3
[   15.802241] Bluetooth: HCI UART protocol H4 registered
[   15.805372] Bluetooth: HCI UART protocol LL registered
[   15.809317] brcmfmac mmc0:0001:1: firmware: failed to load brcm/brcmfmac43455-sdio.raspberrypi,4-compute-module.txt (-2)
[   15.812716] Bluetooth: HCI UART protocol ATH3K registered
[   15.812819] Bluetooth: HCI UART protocol Three-wire (H5) registered
[   15.825001] brcmfmac mmc0:0001:1: firmware: failed to load brcm/brcmfmac43455-sdio.raspberrypi,4-compute-module.txt (-2)
[   15.830030] Bluetooth: HCI UART protocol Intel registered
[   15.844964] brcmfmac mmc0:0001:1: firmware: direct-loading firmware brcm/brcmfmac43455-sdio.txt
[   15.846942] Bluetooth: HCI UART protocol Broadcom registered
[   15.853373] brcmfmac mmc0:0001:1: firmware: failed to load brcm/brcmfmac43455-sdio.raspberrypi,4-compute-module.clm_blob (-2)
[   15.861088] Bluetooth: HCI UART protocol QCA registered
[   15.866842] hci_uart_bcm serial0-0: supply vbat not found, using dummy regulator
[   15.878266] Bluetooth: HCI UART protocol AG6XX registered
[   15.878334] Bluetooth: HCI UART protocol Marvell registered
[   15.885060] hci_uart_bcm serial0-0: supply vddio not found, using dummy regulator
[   15.926548] brcmfmac mmc0:0001:1: firmware: failed to load brcm/brcmfmac43455-sdio.raspberrypi,4-compute-module.clm_blob (-2)
[   15.926942] brcmfmac mmc0:0001:1: firmware: direct-loading firmware brcm/brcmfmac43455-sdio.clm_blob
[   16.047564] uart-pl011 fe201000.serial: no DMA platform data
[   16.332010] Bluetooth: hci0: BCM: chip id 107
[   16.337957] Bluetooth: hci0: BCM: features 0x2f
[   16.345716] Bluetooth: hci0: BCM4345C0
[   16.351305] Bluetooth: hci0: BCM4345C0 (003.001.025) build 0000
[   16.351407] bluetooth hci0: firmware: failed to load brcm/BCM4345C0.raspberrypi,4-compute-module.hcd (-2)
[   16.370373] bluetooth hci0: firmware: failed to load brcm/BCM4345C0.raspberrypi,4-compute-module.hcd (-2)
[   16.371188] bluetooth hci0: firmware: direct-loading firmware brcm/BCM4345C0.hcd
[   16.371711] Bluetooth: hci0: BCM4345C0 'brcm/BCM4345C0.hcd' Patch
```

## 2024-04-30 4B/Trixie install

Headless host, no issues identified. Comparing firmware load messages from `dmesg`

```text
[   13.735062] brcmfmac mmc0:0001:1: firmware: failed to load brcm/brcmfmac43455-sdio.raspberrypi,4-model-b.bin (-2)
[   13.745678] firmware_class: See https://wiki.debian.org/Firmware for information about missing firmware
[   13.750549] bcm2835_v4l2: module is from the staging directory, the quality is unknown, you have been warned.
[   13.755816] brcmfmac mmc0:0001:1: firmware: failed to load brcm/brcmfmac43455-sdio.raspberrypi,4-model-b.bin (-2)
[   13.775815] brcmfmac mmc0:0001:1: Direct firmware load for brcm/brcmfmac43455-sdio.raspberrypi,4-model-b.bin failed with error -2
[   13.788038] usbcore: registered new interface driver brcmfmac
[   13.809305] vc4-drm gpu: bound fe400000.hvs (ops vc4_hvs_ops [vc4])
[   13.818800] brcmfmac mmc0:0001:1: firmware: direct-loading firmware brcm/brcmfmac43455-sdio.bin
[   13.830369] Bluetooth: HCI UART driver ver 2.3
[   13.835507] Bluetooth: HCI UART protocol H4 registered
[   13.842096] brcmfmac mmc0:0001:1: firmware: direct-loading firmware brcm/brcmfmac43455-sdio.raspberrypi,4-model-b.txt
[   13.853028] Registered IR keymap rc-cec
[   13.855707] Bluetooth: HCI UART protocol LL registered
[   13.857047] rc rc0: vc4-hdmi-0 as /devices/platform/soc/fef00700.hdmi/rc/rc0
[   13.862241] Bluetooth: HCI UART protocol ATH3K registered
[   13.870159] brcmfmac mmc0:0001:1: firmware: failed to load brcm/brcmfmac43455-sdio.raspberrypi,4-model-b.clm_blob (-2)
[   13.870511] input: vc4-hdmi-0 as /devices/platform/soc/fef00700.hdmi/rc/rc0/input0
[   13.870816] zfs: module license 'CDDL' taints kernel.
[   13.870822] Disabling lock debugging due to kernel taint
[   13.870875] zfs: module license taints kernel.
[   13.886055] Bluetooth: HCI UART protocol Three-wire (H5) registered
[   13.893947] brcmfmac mmc0:0001:1: firmware: failed to load brcm/brcmfmac43455-sdio.raspberrypi,4-model-b.clm_blob (-2)
[   13.899258] Bluetooth: HCI UART protocol Intel registered
[   13.931900] brcmfmac mmc0:0001:1: firmware: direct-loading firmware brcm/brcmfmac43455-sdio.clm_blob
...
[   14.362517] bluetooth hci0: firmware: failed to load brcm/BCM4345C0.raspberrypi,4-model-b.hcd (-2)
[   14.371733] bluetooth hci0: firmware: failed to load brcm/BCM4345C0.raspberrypi,4-model-b.hcd (-2)
[   14.381645] bluetooth hci0: firmware: direct-loading firmware brcm/BCM4345C0.hcd
[   14.389752] Bluetooth: hci0: BCM4345C0 'brcm/BCM4345C0.hcd' Patch
```
