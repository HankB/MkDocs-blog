# Debian on downstream kernel Part 2

Instructions for running Debian on a Raspberry Pi OS (RpiOS) kernel. See earlier efforts at <https://hankb.github.io/MkDocs-blog/tech/Debian_on_downstream_kernel.01/>

## 2025-10-18 Motivation

Trixie is now Stable and RpiOS has migrated to Trixie. This seems like a good time to give this another try. The description below will repeat much of what was done for the previous iteration so for the sake of brevity, it is going into a new note.

Checking <https://github.com/raspberrypi/linux> the newest stable tag is `stable_20250916` and that will be used. The first try will be using Trixie.

## 2025-10-18 Prep

Pull kernel code and install required build tools.

```text
rm -rf ~/Downloads/pi-kernel/*
mkdir -p ~/Downloads/pi-kernel
cd ~/Downloads/pi-kernel
git clone --depth=1 --branch stable_20250916 https://github.com/raspberrypi/linux
sudo apt install bc bison flex libssl-dev make libc6-dev libncurses5-dev
sudo apt install crossbuild-essential-arm64
```

Set environment variables

```text
CONFIG_LOCALVERSION="-v8-for-Deb-20250916-X"
KERNEL=kernel_2712
```

## 2025-10-18 Build

```text
cd ~/Downloads/pi-kernel/linux
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2712_defconfig
time -p make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs
```

## 2025-10-19 Copy to target media

Back on `olive` and with the NVME SSD in the USB enclosure.

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

Edit the target config.txt to specify the correct kernel (`vmlinuz-6.12.48+deb13-arm64`) and initrd (`initrd.img-6.12.48+deb13-arm64`): In this case thee `kernel=` and `initramfs` lines were already cvorrect, perhaps because RpiOS and Debian are on the same kernel version.

```text
sudo vim mnt/boot/config.txt # or editor of your choice
```

Unmount the target device partitions.

```text
sudo umount mnt/boot
sudo umount mnt/root
```

Install the NVME SSD in the NVME HAT+ on a Pi 5 and power up. No joy. No rainbow screen. It powers up the monitor but the display remains blank.

No boot. Repeating the install and playbooks. Derp. Wrong kernel. New kernel is `kernel_2712.img` and shows as

```text
root@boson:~# uname -a
Linux boson 6.12.47-v8-16k+ #2 SMP PREEMPT Sun Oct 19 16:53:57 CDT 2025 aarch64 GNU/Linux
root@boson:~# 
```

Which seems odd. I thought the V8 kernel was for the 4B and the 2712 for the Pi 5. Now boot from NVME. `Unable to mount root fs ...`. Boot RpiOS from SD card and add `dtparam=pciex1` to `config.txt` with no benefit. Back to USB/NVME and it boots.

It's not clear to me what to do now. Not booting from NVME is a show stopper for me.