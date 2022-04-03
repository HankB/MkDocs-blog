# Firmware issue raspi-firmware Debian Bookworm

<https://github.com/raspberrypi/firmware/issues/1705>  
Forwarded to Debian - https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1007719

## 2022-04-03 test tagged release

Previous testing established that the latest firmware pulled from `master` did not exhibit this problem. That is now pulled into a tagged release `1.2022-328` <https://github.com/raspberrypi/firmware/archive/refs/tags/1.20220328.tar.gz>

I have a Pi 4B/8GH with Bookworm (with `task-gnome-desktop`) installed and with the following kernels:

```text
hbarta@up:~$ ls -l /boot/vmlinuz*
-rw-r--r-- 1 root root 28614528 Dec 18 23:20 /boot/vmlinuz-5.15.0-2-arm64
-rw-r--r-- 1 root root 29998976 Mar 15 06:54 /boot/vmlinuz-5.16.0-5-arm64
-rw-r--r-- 1 root root 30001664 Mar 14 06:04 /boot/vmlinuz-5.17.0-rc8-arm64
-rw-r--r-- 1 root root 30003072 Mar 29 07:16 /boot/vmlinuz-5.17.0-trunk-arm64
hbarta@up:~$ 
```

Testing will be performed with the tagged firmware on all kernels. The following command will be used to determine if the issue exists. 

```text
cat /sys/devices/system/cpu/cpufreq/policy0/scaling_available_frequencies
```

`vcdbg` downloaded from <https://drive.google.com/file/d/1HS9E5vnxxNqrizB4mEYrnFoQQ1axSRKm/view?usp=sharing> is used to confirm firmware version.

Procedure

1. Download and extract the frmware tarball.
1. `cd firmware-1.20220328/boot ` 
1. `cp bootcode.bin *.dat *.elf /boot/firmware`
1. Edit `/boot/firmware/config.txt` to select trhe desired kernel
1. Reboot and check results.

Firmware prior to starting test/

```text
root@up:~# vcdbg version
vcos_build_version     = 40787ee5905644f639a2a0f6e00ae12e517a2211 (clean)
vcos_build_branch      = bcm2711_2
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_application'
vcos_build_application = ?
vcos_build_date        = Aug  3 2021
vcos_build_time        = 18:14:56
vcos_build_user        = dom
vcos_build_hostname    = buildbot
vcos_build_platform    = raspberrypi_linux
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_type'
vcos_build_type        = ?
root@up:~# 
```

## Testing

Attempts to use the older kernels (5.15.6, 5.16.0) were unsuccessful because the installation of one of the more recent kernels apparently purged those. Instead the existing `5.17.0-trunk-arm64` kernel will be tested with the tagged tarball and then testing will be performed with a fresh install and the `raspi-firmware` in the Bookworm repo.

```text
root@up:~# uname -a
Linux up 5.17.0-trunk-arm64 #1 SMP Debian 5.17.1-1~exp1 (2022-03-29) aarch64 GNU/Linux
root@up:~# cat /sys/devices/system/cpu/cpufreq/policy0/scaling_available_frequencies
600000 700000 800000 900000 1000000 1100000 1200000 1300000 1400000 1500000 
root@up:~# vcdbg version
vcos_build_version     = e5a963efa66a1974127860b42e913d2374139ff5 (clean)
vcos_build_branch      = bcm2711_2
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_application'
vcos_build_application = ?
vcos_build_date        = Mar 24 2022
vcos_build_time        = 13:19:26
vcos_build_user        = dom
vcos_build_hostname    = buildbot
vcos_build_platform    = raspberrypi_linux
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_type'
vcos_build_type        = ?
root@up:~# 
```

## Fresh install

```text
xzcat 20220121_raspi_4_bookworm.img.xz> /dev/sdf
```

boot

```text
adduser hbarta
hostnamectl set-hostname charm
apt update
```

reboot, copy `vcdbg`

```text
root@charm:~# uname -a
Linux charm 5.15.0-2-arm64 #1 SMP Debian 5.15.5-2 (2021-12-18) aarch64 GNU/Linux
root@charm:~# cat /sys/devices/system/cpu/cpufreq/policy0/scaling_available_frequencies
600000 700000 800000 900000 1000000 1100000 1200000 1300000 1400000 1500000 
root@charm:~# ./vcdbg version
vcos_build_version     = 40787ee5905644f639a2a0f6e00ae12e517a2211 (clean)
vcos_build_branch      = bcm2711_2
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_application'
vcos_build_application = ?
vcos_build_date        = Aug  3 2021
vcos_build_time        = 18:14:56
vcos_build_user        = dom
vcos_build_hostname    = buildbot
vcos_build_platform    = raspberrypi_linux
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_type'
vcos_build_type        = ?
root@charm:~# 
```

Upgrade `raspi-firmware` only.

```text
root@charm:~# apt install raspi-firmware
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be upgraded:
  raspi-firmware
1 upgraded, 0 newly installed, 0 to remove and 110 not upgraded.
Need to get 4548 kB of archives.
After this operation, 31.7 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bookworm/non-free arm64 raspi-firmware arm64 1.20220120+ds-1 [4548 kB]
Fetched 4548 kB in 2s (2976 kB/s)         
(Reading database ... 18717 files and directories currently installed.)
Preparing to unpack .../raspi-firmware_1.20220120+ds-1_arm64.deb ...
Unpacking raspi-firmware (1.20220120+ds-1) over (1.20210805+ds-1) ...
Setting up raspi-firmware (1.20220120+ds-1) ...

Configuration file '/etc/default/raspi-firmware'
 ==> Modified (by you or by a script) since installation.
 ==> Package distributor has shipped an updated version.
   What would you like to do about it ?  Your options are:
    Y or I  : install the package maintainer's version
    N or O  : keep your currently-installed version
      D     : show the differences between the versions
      Z     : start a shell to examine the situation
 The default action is to keep your current version.
*** raspi-firmware (Y/I/N/O/D/Z) [default=N] ? D
--- /etc/default/raspi-firmware 2022-01-21 08:41:33.325239447 +0000
+++ /etc/default/raspi-firmware.dpkg-new        2022-02-11 05:33:01.000000000 +0000
@@ -25,7 +25,7 @@
 # but you can specify otherwise, including booting by partition label
 # (i.e. ROOTPART="LABEL=root")
 #
-ROOTPART=LABEL=RASPIROOT
+#ROOTPART=/dev/mmcblk0p2
 
 # Main baremetal application that is started by the firmware once the
 # hardware has been initialized. Usually, this is the Linux kernel but
@@ -73,7 +73,7 @@
 # which drives the serial ports) gets its clock from the GPU, as
 # explained here:
 #
-# https://www.raspberrypi.org/documentation/configuration/uart.md
+# https://www.raspberrypi.com/documentation/computers/configuration.html#mini-uart-and-cpu-core-frequency
 #
 # The clock speeds the RPi4 GPU uses are 360/500/550 MHz. If you
 # intend to use the serial console, you need to set GPU_FREQ to
@@ -98,7 +98,7 @@
 
 # Create a file "/etc/default/raspi-firmware-custom" to add custom parameter
 # to startup the kernel. Maybe not all options are supported.
-# (see https://www.raspberrypi.org/documentation/configuration/config-txt/)
+# (see https://www.raspberrypi.com/documentation/computers/config_txt.html)
 #
 # To pass extra arbitrary parameters to the kernel at boot, you can specify
 # them in "/etc/default/raspi-extra-cmdline". Keep in mind they should be

Configuration file '/etc/default/raspi-firmware'
 ==> Modified (by you or by a script) since installation.
 ==> Package distributor has shipped an updated version.
   What would you like to do about it ?  Your options are:
    Y or I  : install the package maintainer's version
    N or O  : keep your currently-installed version
      D     : show the differences between the versions
      Z     : start a shell to examine the situation
 The default action is to keep your current version.
*** raspi-firmware (Y/I/N/O/D/Z) [default=N] ? Y
Installing new version of config file /etc/default/raspi-firmware ...
Installing new version of config file /etc/kernel/postinst.d/z50-raspi-firmware ...
Processing triggers for initramfs-tools (0.140) ...
update-initramfs: Generating /boot/initrd.img-5.15.0-2-arm64
root@charm:~# 
```

Reboot

```text
root@charm:~# uname -a
Linux charm 5.15.0-2-arm64 #1 SMP Debian 5.15.5-2 (2021-12-18) aarch64 GNU/Linux
root@charm:~# cat /sys/devices/system/cpu/cpufreq/policy0/scaling_available_frequencies
cat: /sys/devices/system/cpu/cpufreq/policy0/scaling_available_frequencies: No such file or directory
root@charm:~# dpkg -l | grep raspi-firmware
ii  raspi-firmware             1.20220120+ds-1                   arm64        Raspberry Pi family GPU firmware and bootloaders
root@charm:~# ./vcdbg version 
vcos_build_version     = bd88f66f8952d34e4e0613a85c7a6d3da49e13e2 (clean)
vcos_build_branch      = bcm2711_2
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_application'
vcos_build_application = ?
vcos_build_date        = Jan 20 2022
vcos_build_time        = 13:56:48
vcos_build_user        = dom
vcos_build_hostname    = buildbot
vcos_build_platform    = raspberrypi_linux
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_type'
vcos_build_type        = ?
root@charm:~# 
```

Wrong package.


```text
wget http://ftp.us.debian.org/debian/pool/non-free/r/raspi-firmware/raspi-firmware_1.20220328+ds-1_arm64.deb
```

```text
root@charm:~# apt install ./raspi-firmware_1.20220328+ds-1_arm64.deb 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Note, selecting 'raspi-firmware' instead of './raspi-firmware_1.20220328+ds-1_arm64.deb'
The following packages will be upgraded:
  raspi-firmware
1 upgraded, 0 newly installed, 0 to remove and 110 not upgraded.
Need to get 0 B/4551 kB of archives.
After this operation, 8192 B of additional disk space will be used.
Get:1 /root/raspi-firmware_1.20220328+ds-1_arm64.deb raspi-firmware arm64 1.20220328+ds-1 [4551 kB]
(Reading database ... 18717 files and directories currently installed.)
Preparing to unpack .../raspi-firmware_1.20220328+ds-1_arm64.deb ...
Unpacking raspi-firmware (1.20220328+ds-1) over (1.20220120+ds-1) ...
Setting up raspi-firmware (1.20220328+ds-1) ...
Processing triggers for initramfs-tools (0.140) ...
update-initramfs: Generating /boot/initrd.img-5.15.0-2-arm64
N: Download is performed unsandboxed as root as file '/root/raspi-firmware_1.20220328+ds-1_arm64.deb' couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)
root@charm:~# 
```

reboot

```text
hbarta@charm:~$ uname -a
Linux charm 5.15.0-2-arm64 #1 SMP Debian 5.15.5-2 (2021-12-18) aarch64 GNU/Linux
hbarta@charm:~$ cat /sys/devices/system/cpu/cpufreq/policy0/scaling_available_frequencies
600000 700000 800000 900000 1000000 1100000 1200000 1300000 1400000 1500000 
hbarta@charm:~$ su - 
root@charm:~# ./vcdbg version
vcos_build_version     = e5a963efa66a1974127860b42e913d2374139ff5 (clean)
vcos_build_branch      = bcm2711_2
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_application'
vcos_build_application = ?
vcos_build_date        = Mar 24 2022
vcos_build_time        = 13:19:26
vcos_build_user        = dom
vcos_build_hostname    = buildbot
vcos_build_platform    = raspberrypi_linux
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_type'
vcos_build_type        = ?
root@charm:~# 
```

Perform full upgrade and reboot

```text
110 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
```

```text
hbarta@charm:~$ uname -a
Linux charm 5.16.0-5-arm64 #1 SMP Debian 5.16.14-1 (2022-03-15) aarch64 GNU/Linux
hbarta@charm:~$ cat /sys/devices/system/cpu/cpufreq/policy0/scaling_available_frequencies
600000 700000 800000 900000 1000000 1100000 1200000 1300000 1400000 1500000 
hbarta@charm:~$ su -
root@charm:~# ./vcdbg version
vcos_build_version     = e5a963efa66a1974127860b42e913d2374139ff5 (clean)
vcos_build_branch      = bcm2711_2
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_application'
vcos_build_application = ?
vcos_build_date        = Mar 24 2022
vcos_build_time        = 13:19:26
vcos_build_user        = dom
vcos_build_hostname    = buildbot
vcos_build_platform    = raspberrypi_linux
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_type'
vcos_build_type        = ?
root@charm:~#
```

Install with kernel from experimental

```text
root@charm:~# cat /etc/apt/preferences.d/linux-kernel
Package: *
Pin: release o=Debian,a=experimental
Pin-Priority: 102
root@up:~# apt policy
Package files:
 100 /var/lib/dpkg/status
     release a=now
 500 http://deb.debian.org/debian bookworm/non-free arm64 Packages
     release o=Debian,a=testing,n=bookworm,l=Debian,c=non-free,b=arm64
     origin deb.debian.org
 500 http://deb.debian.org/debian bookworm/contrib arm64 Packages
     release o=Debian,a=testing,n=bookworm,l=Debian,c=contrib,b=arm64
     origin deb.debian.org
 500 http://deb.debian.org/debian bookworm/main arm64 Packages
     release o=Debian,a=testing,n=bookworm,l=Debian,c=main,b=arm64
     origin deb.debian.org
Pinned packages:

root@charm:~#
root@charm:~# 
root@charm:~# echo "deb http://deb.debian.org/debian experimental main" >> /etc/apt/sources.list
root@charm:~# apt update
Hit:1 http://deb.debian.org/debian bookworm InRelease                               
Hit:2 http://security.debian.org/debian-security bookworm-security InRelease        
Get:3 http://deb.debian.org/debian experimental InRelease [75.4 kB]
Get:4 http://deb.debian.org/debian experimental/main arm64 Packages [369 kB]
Get:5 http://deb.debian.org/debian experimental/main Translation-en [232 kB]
Fetched 677 kB in 3s (250 kB/s)        
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
root@charm:~# apt install -t experimental linux-image-5.17.0-trunk-arm64                                     
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Suggested packages:
  linux-doc-5.17 debian-kernel-handbook
The following NEW packages will be installed:
  linux-image-5.17.0-trunk-arm64
0 upgraded, 1 newly installed, 0 to remove and 16 not upgraded.
Need to get 58.6 MB of archives.
After this operation, 406 MB of additional disk space will be used.
Get:1 http://deb.debian.org/debian experimental/main arm64 linux-image-5.17.0-trunk-arm64 arm64 5.17.1-1~exp1 [58.6 MB]
Fetched 58.6 MB in 4s (13.8 MB/s)                         
Selecting previously unselected package linux-image-5.17.0-trunk-arm64.
(Reading database ... 23590 files and directories currently installed.)
Preparing to unpack .../linux-image-5.17.0-trunk-arm64_5.17.1-1~exp1_arm64.deb ...
Unpacking linux-image-5.17.0-trunk-arm64 (5.17.1-1~exp1) ...
Setting up linux-image-5.17.0-trunk-arm64 (5.17.1-1~exp1) ...
I: /vmlinuz.old is now a symlink to boot/vmlinuz-5.16.0-5-arm64
I: /initrd.img.old is now a symlink to boot/initrd.img-5.16.0-5-arm64
I: /vmlinuz is now a symlink to boot/vmlinuz-5.17.0-trunk-arm64
I: /initrd.img is now a symlink to boot/initrd.img-5.17.0-trunk-arm64
/etc/kernel/postinst.d/initramfs-tools:
update-initramfs: Generating /boot/initrd.img-5.17.0-trunk-arm64
root@charm:~# 
```

Note: Results of following `apt policy` command were inadvertently copied into `/etc/apt/preferences.d/linux-kernel` but seem to be ignored by `apt` or at least did not prevent the desired operation.

Reboot

```text
hbarta@charm:~$ uname -a
Linux charm 5.17.0-trunk-arm64 #1 SMP Debian 5.17.1-1~exp1 (2022-03-29) aarch64 GNU/Linux
hbarta@charm:~$ cat /sys/devices/system/cpu/cpufreq/policy0/scaling_available_frequencies
600000 700000 800000 900000 1000000 1100000 1200000 1300000 1400000 1500000 
hbarta@charm:~$ su -
root@charm:~# ./vcdbg version
vcos_build_version     = e5a963efa66a1974127860b42e913d2374139ff5 (clean)
vcos_build_branch      = bcm2711_2
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_application'
vcos_build_application = ?
vcos_build_date        = Mar 24 2022
vcos_build_time        = 13:19:26
vcos_build_user        = dom
vcos_build_hostname    = buildbot
vcos_build_platform    = raspberrypi_linux
debug_sym: ReadVideoCoreMemoryBySymbol: Symbol not found: 'vcos_build_type'
vcos_build_type        = ?
root@charm:~# 
```
