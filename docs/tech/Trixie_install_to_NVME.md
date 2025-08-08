# Trixie install to NVME

## 2025-08-08 Motivation

Provide detailed instructions for installing the Trixie image from <https://raspi.debian.net/tested-images/> to an NVME SSD without unnecessarily cluttering the instructions for running [Debian on the Downstream kernel](./Debian_on_downstream_kernel.md).

## 2025-08-08 Overview

Use the Ansible playbooks at <https://github.com/HankB/polana-ansible> to install and configure a bootable (on a Pi 4B) Trixie system.

## 2025-08-08 Provisioning

Copy previously downloaded image to the target media and tweak some settings. The SSD in the adapter/enclosure appears as `/dev/sdb`. This step is performed on an X86_64 host but could also be done on a Raspberry Pi. If done on a CM4 booted from SD and with the NVME SSD in a PCIe adapter, the `erase SSD (blkdiscard)` step should succeed. With a USB connected SSD it may fail and failure is ignored.

```text
ansible-playbook provision-Debian-lite.yml -b -K --extra-vars "ssd_dev=/dev/sdb \
    os_image=/home/hbarta/Downloads/Pi/Debian/20231111_raspi_4_trixie.img.xz \
    new_host_name=kweli part_prefix=\"\" \
    eth_spoof_mac=dc:a6:32:bf:65:b5 wifi_spoof_mac=dc:a6:32:bf:65:b4"
```

## 2025-08-08 First boot

No boot. :-/ Steps requiring a running system are deferred until the [upstream kernel is installed](./Debian_on_downstream_kernel.md#2025-08-08-copy-to-target-media) and the system boots from the target media on a Pi 5.

## 2025-08-08 Continue Setup

Identify the IP address or hostname for the Pi 5 and edit `inventory` accordingly. Confirm that `ssh root@[pi 5 host]` works.

```text
ansible-playbook first-boot-Debian.yml -i inventory -u root
```

[to be continued]
