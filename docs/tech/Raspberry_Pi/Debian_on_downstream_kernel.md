# Debian on downstream kernel

Instructions for running Debian on a Raspberry Pi OS (RpiOS) kernel.

## 2025-08-08 Motivation

Debian is my distro of choice (after using various distros for decades.) The Raspberry Pi 5 has as new I/O subsystem based on the RP1 chip and which requires significant S/W modifications to run Linux. These have not yet been fully upstreamed such that Debian can boot and run on a Pi 5 (as of kernel version 6.16.)

## 2025-08-08 Overview

1. Install Debian using a Pi 4B (or CM4) on the target storage. This example will use an NVME SSD. A previous proof-of-concept was tested using a USB connected SATA SSD.
1. Build the kernel per instructions kindly provided at <https://www.raspberrypi.com/documentation/computers/linux_kernel.html>. This can be done on the Pi 44B/CM4 (hours) or cross built on a more powerful host (minutes.)
1. Install the kernel, modules and related files (DTBs) to the target media.
1. Boot the Pi 5 from the target media.

## 2025-08-08 Details

The desired target media is an NVME SSD and the development platform is a Ryzen 7 7700X based host running Debian Bookworm. The storage will be connected to the Ryzen host using a USB/NVME housing and to the Pi 5 using the Waveshare NVME HAT+. The target OS will be Debian Trixie (which will see release as Stable tomorrow.) The most recent kernel branch at <https://github.com/raspberrypi/linux> is `rpi-6.12.y` and that will be used for thir process.

## 2025-08-08 Prep

Pull kernel code and install required build tools.

```text
mkdir -p ~/Downloads/kernel
cd ~/Downloads/kernel
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

Edit the target config.txt to spedify the correct kernel:

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

## notes

* [^1] @ukleinek on IRC suggested `make bindeb-pkg` to produce a .deb for the modules. This will be convenient when building a new kernel on the Pi 5 but is less so when cross developing on another host.