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

## 2025-08-09 second post boot playbook

Put the 2242 NVME SSD in the Pi 5 and booted from an SD card running RpiOS. Edited the `config.txt` in the SSD to reflect the correct kernel. The host then booted the Debian install on the RpiOS kernel. Next logged in using `root@kweli` and performed an `apt update`. The list of updates included a kernel so that was held `apt-mark hold ...`. The subsequent update still overwrote `config.txt` so that was modified again. Perhaps following the instructions in the file

```text
# If you need to set boot-time parameters, do so via the
# /etc/default/raspi-firmware, /etc/default/raspi-firmware-custom or
# /etc/default/raspi-extra-cmdline files.
```

Running the second post boot playbook. (By mistake, the one that builds the ZFS modules - it will be interesting to see if that works. It did not) 

```text
ansible-playbook second-boot-bookworm-Debian.yml -i inventory -u root
ansible-playbook second-boot-bookworm-lite-Debian.yml -i inventory -u root
```

Edit `/etc/default/raspi-firmware-custom` and fixum `config.txt` and subsequent boot halts on `pinctrl_lookup_state`. Added `dtparam=pciex1` to config.txt with same result.

## 2025-08-13 repeat install

To 256GB 2242 NVME in USB enclosure on `olive`

```text
ansible-playbook provision-Debian-lite.yml -v -b -K --extra-vars "ssd_dev=/dev/sdb \
    os_image=/home/hbarta/Downloads/Pi/Debian/20231111_raspi_4_trixie.img.xz \
    new_host_name=kweli part_prefix=\"\" \
    eth_spoof_mac=dc:a6:32:bf:65:b5 wifi_spoof_mac=dc:a6:32:bf:65:b4"
```

Remove from enclosure and install in CM4 for first boot. Boot fails on NVME Timeout. Back in the USB enclosure and boot in Pi 4B. Rin first boot playbook.

```text
ansible-playbook first-boot-Debian.yml -i inventory -u root
```

Second boot (lite) playbook. (No ZFS modules.) Ran the regular playbook by mistake which failed when tryint to import the (non-existent) pool. Ran the regular playbook to complete setup.

```text
ansible-playbook second-boot-bookworm-lite-Debian.yml -i inventory -u root
```

Login to Pi 4B and update/upgrade to Sid. 107 packages to upgrade. But first purge `zfs-dkms` and zfsutils-linux`.

```text
apt purge zfs-dkms zfsutils-linux
apt autoremove
apt upgrade
```

Host boots. Switch back to [Debian on downstream kernel](./Debian_on_downstream_kernel.md#2025-08-13-build).
