# Debian on downstream kernel

Instructions for running Debian on a Raspberry Pi OS (RpiOS) kernel.

## 2025-08-08 Motivation

Debian is my distro of choice (after using various distros for decades.) The Raspberry Pi 5 has a new I/O subsystem based on the RP1 chip and which requires significant S/W modifications to run Linux. These have not yet been fully upstreamed such that Debian can boot and run on a Pi 5 (as of kernel version 6.16.)

## 2025-08-08 Overview

1. Install Debian using a Pi 4B (or CM4) on the target storage. This example will use an NVME SSD. A previous proof-of-concept was tested using a USB connected SATA SSD.
1. Build the kernel per instructions kindly provided at <https://www.raspberrypi.com/documentation/computers/linux_kernel.html>. This can be done on the Pi 4B/CM4 (hours) or cross built on a more powerful host (minutes.)
1. Install the kernel, modules and related files (DTBs) to the target media.
1. Boot the Pi 5 from the target media.

## 2025-08-08 Details

The desired target media is an NVME SSD and the development platform is a Ryzen 7 7700X based host running Debian Bookworm. The storage will be connected to the Ryzen host using a USB/NVME housing and to the Pi 5 using the Waveshare NVME HAT+. The target OS will be Debian Trixie (which will see release as Stable tomorrow.) The most recent kernel branch at <https://github.com/raspberrypi/linux> is `rpi-6.12.y` and that will be used for this process.

## 2025-08-08 Prep

Pull kernel code and install required build tools.

```text
mkdir -p ~/Downloads/pi-kernel
cd ~/Downloads/pi-kernel
git clone --depth=1 --branch rpi-6.12.y https://github.com/raspberrypi/linux
sudo apt install bc bison flex libssl-dev make libc6-dev libncurses5-dev
sudo apt install crossbuild-essential-arm64
```

Set environment variables

```text
CONFIG_LOCALVERSION="-v8-for-Deb-X"
KERNEL=kernel_2712
```

## 2025-08-08 Build [^1]

```text
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2712_defconfig
time -p make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs
```

## 2025-08-08 Prepare target

[Using Ansible playbooks](./Trixie_install_to_NVME.md). The result did not boot from a CM4. This procedure will proceed with the target as provisioned and prior to the first boot. Any preferred method of provisioning the target media should work.

## 2025-08-08 Copy to target media

Identify the device, in this case `/dev/sdb` and mount accordingly:

```text
cd ~/Downloads/pi-kernel/linux
mkdir -p mnt/boot
mkdir -p mnt/root
sudo mount /dev/sdb1 mnt/boot
sudo mount /dev/sdb2 mnt/root
sudo mkdir -p mnt/boot/overlays

```

Install modules to target:

```text
sudo env PATH=$PATH make -j1$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=mnt/root modules_install
```

Install the kernel

```text
sudo cp mnt/boot/$KERNEL.img mnt/boot/$KERNEL-backup.img
sudo cp arch/arm64/boot/Image mnt/boot/$KERNEL.img
sudo cp arch/arm64/boot/dts/broadcom/*.dtb mnt/boot/
sudo cp arch/arm64/boot/dts/overlays/*.dtb* mnt/boot/overlays/
sudo cp arch/arm64/boot/dts/overlays/README mnt/boot/overlays/
```

Edit the target config.txt to specify the correct kernel:

```text
sudo vim mnt/boot/config.txt # or editor of your choice
```

Unmount the target device partitions.

```text
sudo umount mnt/boot
sudo umount mnt/root
```

Install the NVME SSD in the NVME HAT_ and boot the system. (Great Success!) At this point the process is complete and the user can proceed however they wish to complete the installation

Further customization is peformed [using Ansible Playbooks](./Trixie_install_to_NVME.md#2025-08-08-continue-setup).

Note: Following first boot playbook the host no longer boots. This requires further investigation.

## 2025-08-12 try using Sid

### Plan

1. Install the [Trixie image](https://raspi.debian.net/tested/20231111_raspi_4_trixie.img.xz) to an NVME SSD (in a USB enclosure) (Using Ansible playbook.)
1. Move the SSD to a CM4 with PCIe/NVME board.
1. Boot and upgrade using Ansible playbook and complete configuration.
1. Edit `sources.list` and upgrade to Sid.
1. Return card to USB enclosure and install downstream kernel.
1. Install card in Pi 5 and boot.

## 2025-08-13 build newer kernel

Checking out <https://github.com/raspberrypi/linux/tags> it seems that the branch I was using is old. It seems like the most recent stable tag is worth a try as well.

```text
mkdir -p ~/Downloads/pi-kernel
cd ~/Downloads/pi-kernel
time -p git clone --branch stable_20250702 --depth=1 https://github.com/raspberrypi/linux
```

Set environment variables

```text
CONFIG_LOCALVERSION="-v8-for-Deb-20250702-X"
KERNEL=kernel_2712
```

## 2025-08-13 Build

```text
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2712_defconfig
time -p make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs
```

Follow [Sid install and configuration](./Trixie_install_to_NVME.md#2025-08-13-repeat-install)
Install using USB enclosure on `X86_64` host (`olive`.)

## 2025-08-13 Copy to target media
Return the USB/NVME enclosure to the cross build host. Identify the device, in this case `/dev/sdb` and mount accordingly:

```text
cd ~/Downloads/pi-kernel/linux
mkdir -p mnt/boot
mkdir -p mnt/root
sudo mount /dev/sdb1 mnt/boot
sudo mount /dev/sdb2 mnt/root
sudo mkdir -p mnt/boot/overlays

```

Install modules to target:

```text
sudo env PATH=$PATH make -j1$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=mnt/root modules_install
```

Install the kernel

```text
sudo cp mnt/boot/$KERNEL.img mnt/boot/$KERNEL-backup.img
sudo cp arch/arm64/boot/Image mnt/boot/$KERNEL.img
sudo cp arch/arm64/boot/dts/broadcom/*.dtb mnt/boot/
sudo cp arch/arm64/boot/dts/overlays/*.dtb* mnt/boot/overlays/
sudo cp arch/arm64/boot/dts/overlays/README mnt/boot/overlays/
```

Edit the target config.txt to specify the correct kernel:

```text
sudo vim mnt/boot/config.txt # or editor of your choice
```

Unmount the target device partitions.

```text
sudo umount mnt/boot
sudo umount mnt/root
```

Move the USB enclosure to the Pi 5 and boot. Host comes up.

Comment out `upstream_kernel=1` in config.txt and reboot. Host comes up and is running:

```text
hbarta@kweli:~$ uname -a
Linux kweli 6.12.34-v8-16k+ #2 SMP PREEMPT Wed Aug 13 10:59:11 CDT 2025 aarch64 GNU/Linux
hbarta@kweli:~$ 
```

Add `dtparam=pciex1` and consider adding `dtparam=pciex1_gen=3`. Shutdown and move SSD to NVME HAT. Pi 5 boots.

Heed the warning in `config.txt`:

```text
# If you need to set boot-time parameters, do so via the
# /etc/default/raspi-firmware, /etc/default/raspi-firmware-custom or
# /etc/default/raspi-extra-cmdline files.
```

```text
vim /etc/default/raspi-firmware /etc/default/raspi-firmware-custom
```

Upgrade (one package `parted`) and reboot. Now go for KDE/Plasma 

```text
apt install kde-plasma-desktop # 1329 packages.
```

Completes and on checking `config.txt` I find

```text
root@kweli:~# cat /boot/firmware/config.txt 
# Do not modify this file!
#
# It is automatically generated upon install or update of either the
# firmware or the Linux kernel.
#
# If you need to set boot-time parameters, do so via the
# /etc/default/raspi-firmware, /etc/default/raspi-firmware-custom or
# /etc/default/raspi-extra-cmdline files.

# Switch the CPU from ARMv7 into ARMv8 (aarch64) mode
arm_64bit=1

enable_uart=1
upstream_kernel=1

kernel=vmlinuz-6.12.41+deb13-arm64
# For details on the initramfs directive, see
# https://www.raspberrypi.org/forums/viewtopic.php?f=63&t=10532
initramfs initrd.img-6.12.41+deb13-arm64

# Inserted by /etc/default/raspi-firmware-custom

# raspi-firmware-custom
dtparam=pciex1
kernel=kernel_2712.img
root@kweli:~# 
```

I will add `upstream_kernel=0` to `/etc/default/raspi-firmware-custom` (and edit it that way now.) `systemctl start sddm` did not work. No login screen.

```text
root@kweli:~# systemctl start sddm
root@kweli:~# systemctl status sddm
● sddm.service - Simple Desktop Display Manager
     Loaded: loaded (/usr/lib/systemd/system/sddm.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-08-13 12:00:41 CDT; 11s ago
 Invocation: 61ed8c0fb0bc4c01a9148618a7860c4f
       Docs: man:sddm(1)
             man:sddm.conf(5)
   Main PID: 33731 (sddm)
      Tasks: 2 (limit: 9549)
        CPU: 43ms
     CGroup: /system.slice/sddm.service
             └─33731 /usr/bin/sddm

Aug 13 12:00:41 kweli systemd[1]: Started sddm.service - Simple Desktop Display Manager.
Aug 13 12:00:41 kweli sddm[33731]: Initializing...
Aug 13 12:00:41 kweli sddm[33731]: Starting...
Aug 13 12:00:41 kweli sddm[33731]: Logind interface found
root@kweli:~# 
```

No joy on `reboot`. Fails on `pinctrl_lookup_state`.

## notes

* [^1] @ukleinek on IRC suggested `make bindeb-pkg` to produce a .deb for the modules. This will be convenient when building a new kernel on the Pi 5 but is less so when cross developing on another host.