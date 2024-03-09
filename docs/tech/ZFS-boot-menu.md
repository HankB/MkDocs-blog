# ZFS boot menu

Perform another trial before attempting to rescue `oak`. Installing on the 128GB Kingston SSD on `oak` H/W (other drives disconnected) and booting the live Debian 12 install w/ persistence and following instructions at <https://docs.zfsbootmenu.org/en/v2.3.x/guides/debian/bookworm-uefi.html> 

```text
ssh user@192.168.1.80 # passwd = live
```

## 2024-03-08 Configure Live Environment

No EFI vars, as expected., Apt already configured and ZFS installed.

```text
root@debian:~# cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
root@debian:~# 

root@debian:~# cat /etc/apt/sources.list
deb http://deb.debian.org/debian/ bookworm main non-free-firmware contrib non-free
deb-src http://deb.debian.org/debian/ bookworm main non-free-firmware contrib non-free
deb [trusted=yes] file:/run/live/medium bookworm main non-free-firmware
root@debian:~# 

root@debian:~# zfs --version
zfs-2.1.11-1
zfs-kmod-2.1.11-1
root@debian:~# 
```

## 2024-03-08 Define disk variables

variables for **boot files**

```text
export BOOT_DISK="/dev/sda"
export BOOT_PART="1"
export BOOT_DEVICE="${BOOT_DISK}${BOOT_PART}"
```

variables for **ZFS pool**

```text
export POOL_DISK="/dev/sda"
export POOL_PART="2"
export POOL_DEVICE="${POOL_DISK}${POOL_PART}"
```

## 2024-03-08 Disk preparation

### Wipe partitions

Wipe the SSD After confirming that `/dev/sda` is in fact the Kingston 128GB SSD.

```text
root@debian:~# blkdiscard -f /dev/sda
blkdiscard: Operation forced, data will be lost!
root@debian:~# 
```

### Create EFI boot partition

Not used for this install, but perhaps in the future

```text
sgdisk -n "${BOOT_PART}:1m:+512m" -t "${BOOT_PART}:ef00" "$BOOT_DISK"
```

```text
root@debian:~# sgdisk -n "${BOOT_PART}:1m:+512m" -t "${BOOT_PART}:ef00" "$BOOT_DISK"
Creating new GPT entries in memory.
The operation has completed successfully.
root@debian:~# sgdisk -p /dev/sda
Disk /dev/sda: 234441648 sectors, 111.8 GiB
Model: KINGSTON SA400S3
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 7B794CDC-A742-48C3-BDC2-3825E3E776DD
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 234441614
Partitions will be aligned on 2048-sector boundaries
Total free space is 233393005 sectors (111.3 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         1050623   512.0 MiB   EF00  
root@debian:~# 
```

### Create zpool partition

```text
sgdisk -n "${POOL_PART}:0:-10m" -t "${POOL_PART}:bf00" "$POOL_DISK"
```

```text
root@debian:~# sgdisk -n "${POOL_PART}:0:-10m" -t "${POOL_PART}:bf00" "$POOL_DISK"
The operation has completed successfully.
root@debian:~# sgdisk -p /dev/sda
Disk /dev/sda: 234441648 sectors, 111.8 GiB
Model: KINGSTON SA400S3
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 7B794CDC-A742-48C3-BDC2-3825E3E776DD
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 234441614
Partitions will be aligned on 2048-sector boundaries
Total free space is 22494 sectors (11.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         1050623   512.0 MiB   EF00  
   2         1050624       234421134   111.3 GiB   BF00  
root@debian:~# ```

## 2024-03-08 ZFS pool creation

### Create the zpool

```text
zpool create -f -o ashift=12 \
 -O compression=lz4 \
 -O acltype=posixacl \
 -O xattr=sa \
 -O relatime=on \
 -o autotrim=off \
 -o compatibility=openzfs-2.1-linux \
 -m none zroot "$POOL_DEVICE"
```

```text
root@debian:~# zpool create -f -o ashift=12 \
 -O compression=lz4 \
 -O acltype=posixacl \
 -O xattr=sa \
 -O relatime=on \
 -o autotrim=off \
 -o compatibility=openzfs-2.1-linux \
 -m none zroot "$POOL_DEVICE"
root@debian:~# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zroot   111G   504K   111G        -         -     0%     0%  1.00x    ONLINE  -
root@debian:~# 
```

### Create initial file systems

```text
zfs create -o mountpoint=none zroot/ROOT
zfs create -o mountpoint=/ -o canmount=noauto zroot/ROOT/${ID}
zfs create -o mountpoint=/home zroot/home

export export ID=bookworm
zpool set bootfs=zroot/ROOT/${ID} zroot
```

```text
root@debian:~# zfs create -o mountpoint=none zroot/ROOT
zfs create -o mountpoint=/ -o canmount=noauto zroot/ROOT/${ID}
zfs create -o mountpoint=/home zroot/home

root@debian:~# zpool set bootfs=zroot/ROOT/bookworm zroot
cannot set property for 'zroot': no such pool or dataset
root@debian:~# zpool get bootfs zroot
NAME   PROPERTY  VALUE   SOURCE
zroot  bootfs    -       default
root@debian:~# 
```

### Export, then re-import with a temporary mountpoint of `/mnt`

```text
zpool export zroot
zpool import -N -R /mnt zroot
zfs mount zroot/ROOT/${ID}
zfs mount zroot/home
```

### Verify that everything is mounted correctly

```text
root@debian:~# zpool export zroot
root@debian:~# zpool import -N -R /mnt zroot
root@debian:~# zfs mount zroot/ROOT/${ID}
root@debian:~# zfs mount zroot/home
root@debian:~# mount | grep mnt
zroot/ROOT/bookworm on /mnt type zfs (rw,relatime,xattr,posixacl)
zroot/home on /mnt/home type zfs (rw,relatime,xattr,posixacl)
root@debian:~# 
```

### Update device symlinks

```text
udevadm trigger
```

### Install Debian

```text
time -p debootstrap bookworm /mnt
```

```text
root@debian:~# time -p debootstrap bookworm /mnt
I: Target architecture can be executed
I: Retrieving InRelease 
I: Checking Release signature
I: Valid Release signature (key id 4D64FEC119C2029067D6E791F8D2585B8783D481)
I: Retrieving Packages 
...
I: Base system installed successfully.
real 105.99
user 70.91
sys 24.98
root@debian:~# 
```

### Copy files into the new install

```text
cp /etc/hostid /mnt/etc
cp /etc/resolv.conf /mnt/etc
```

### Chroot into the new OS

```text
mount -t proc proc /mnt/proc
mount -t sysfs sys /mnt/sys
mount -B /dev /mnt/dev
mount -t devpts pts /mnt/dev/pts
chroot /mnt /bin/bash
```

### Set a hostname

```text
export YOURHOSTNAME=oakzbm
echo ${YOURHOSTNAME} > /etc/hostname
echo -e "127.0.1.1\t${YOURHOSTNAME}" >> /etc/hosts
```

```text
root@debian:/# cat /etc/hostname
oakzbm
root@debian:/# cat /etc/hosts
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

127.0.1.1       oakzbm
root@debian:/# 
```

### Set a root password

```text
passwd
```

### configure `apt`

NB Add firmware entry.

```text
cat <<EOF > /etc/apt/sources.list
deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware

deb http://deb.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian-security/ bookworm-security main contrib non-free non-free-firmware

deb http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware

deb http://deb.debian.org/debian bookworm-backports main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian bookworm-backports main contrib non-free non-free-firmware
EOF
```

```text
root@debian:/# cat /etc/apt/sources.list
deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware

deb http://deb.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian-security/ bookworm-security main contrib non-free non-free-firmware

deb http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware

deb http://deb.debian.org/debian bookworm-backports main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian bookworm-backports main contrib non-free non-free-firmware
root@debian:/# 
```

### Update the repository cache

```text
apt update
```

### Install additional base packages

```text
apt install -y locales keyboard-configuration console-setup
```

### Configure packages to customize local and console properties

```text
dpkg-reconfigure locales tzdata keyboard-configuration console-setup
```

## 2024-03-08 ZFS Configuration

### Install required packages

```text
apt install linux-headers-amd64 linux-image-amd64 zfs-initramfs dosfstools
echo "REMAKE_INITRD=yes" > /etc/dkms/zfs.conf
```

### Enable systemd ZFS services

```text
systemctl enable zfs.target
systemctl enable zfs-import-cache
systemctl enable zfs-mount
systemctl enable zfs-import.target
```

### Configure initramfs-tools

(No required steps)

### Rebuild the initramfs

```text
update-initramfs -c -k all
```

## 2024-03-08 Install and configure ZFSBootMenu

### Set ZFSBootMenu properties on datasets

```text
zfs set org.zfsbootmenu:commandline="quiet" zroot/ROOT
```

## 2024-03-08 Install and configure syslinux

Need to review instructions at https://docs.zfsbootmenu.org/en/v2.3.x/guides/void-linux/syslinux-mbr.html#install-and-configure-syslinux

### Create a ext4 boot filesystem

```text
mkfs.ext4 -O '^64bit' "$BOOT_DEVICE"
```

### Create an fstab entry and mount

```text
cat << EOF >> /etc/fstab
$( blkid | grep "$BOOT_DEVICE" | cut -d ' ' -f 2 ) /boot/syslinux ext4 defaults 0 0
EOF

mkdir /boot/syslinux
mount /boot/syslinux
```

```text
root@debian:/# df /boot/syslinux
Filesystem     1K-blocks  Used Available Use% Mounted on
/dev/sda1         499284    24    462564   1% /boot/syslinux
root@debian:/# 
```

### Install the syslinux package, copy modules

```text
apt install -y syslinux extlinux
cp /usr/lib/syslinux/modules/bios/*.c32 /boot/syslinux
```

```text
root@debian:/# find /boot/syslinux | wc -l
62
root@debian:/# 
```

### Install extlinux

```text
extlinux --install /boot/syslinux
```

### Install the syslinux MBR data

```text
dd bs=440 count=1 conv=notrunc if=/usr/lib/syslinux/mbr/mbr.bin of="$BOOT_DISK"
```

```text
root@debian:/# dd bs=440 count=1 conv=notrunc if=/usr/lib/syslinux/mbr/mbr.bin of="$BOOT_DISK"
1+0 records in
1+0 records out
440 bytes copied, 0.000356405 s, 1.2 MB/s
root@debian:/# 
```

## 2024-03-08 Install and configure ZFSBootMenu

### Set ZFSBootMenu properties on datasets

(Already done above)

### Install the ZFSBootMenu package

```text
wget get.zfsboot.menu/latest.tar.gz
```

Deposits a file `latest.tar.gz` that unpacks to 

```text
root@debian:/# tar tf latest.tar.gz
zfsbootmenu-release-x86_64-v2.3.0/
zfsbootmenu-release-x86_64-v2.3.0/vmlinuz-bootmenu
zfsbootmenu-release-x86_64-v2.3.0/initramfs-bootmenu.img
root@debian:/# 
```

(Need to sort out the signature verification at <https://docs.zfsbootmenu.org/en/v2.3.x/#signature-verification-and-prebuilt-efi-executables>)

```text
mkdir -p /boot/syslinux/zfsbootmenu/
cp zfsbootmenu-release-x86_64-v2.3.0/* /boot/syslinux/zfsbootmenu/
```

### Enable zfsbootmenu image creation

Edit `/etc/zfsbootmenu/config.yaml` and make sure that the following parameters are set:

```text
Global:
  ManageImages: true
  BootMountPoint: /boot/syslinux
Components:
  Enabled: true
  Versions: false
  ImageDir: /boot/syslinux/zfsbootmenu
```

```text
mkdir -p /etc/zfsbootmenu/
cat <<EOF > /etc/zfsbootmenu/config.yaml
Global:
  ManageImages: true
  BootMountPoint: /boot/syslinux
Components:
  Enabled: true
  Versions: false
  ImageDir: /boot/syslinux/zfsbootmenu
EOF
```

### Configure syslinux

```text
cat > /boot/syslinux/syslinux.cfg <<EOF
UI menu.c32
PROMPT 0

MENU TITLE ZFSBootMenu
TIMEOUT 50

DEFAULT zfsbootmenu

LABEL zfsbootmenu
  MENU LABEL ZFSBootMenu
  KERNEL /zfsbootmenu/vmlinuz-bootmenu
  INITRD /zfsbootmenu/initramfs-bootmenu.img
  APPEND zfsbootmenu quiet

LABEL zfsbootmenu-backup
  MENU LABEL ZFSBootMenu (Backup)
  KERNEL /zfsbootmenu/vmlinuz-bootmenu-backup
  INITRD /zfsbootmenu/initramfs-bootmenu-backup.img
  APPEND zfsbootmenu quiet
EOF
```

### Generate the initial ZFSBootMenu initramfs

This is where things got murky. I tried using `update-initramfs -c -k all` to generate the initrd but it's not clear to me that this is needed. And the command failed anyway. I copied the files from the `latest.tar.gz` to `/boot` where `update-initramfs` would find them. On the first try it complained about not finding `/boot/config-bootmenu`. The corresponding one for the running kernel included the build config settings so I created an empty config file `touch /boot/config-bootmenu`. the next complaint was about missing compression:

```text
root@debian:/# update-initramfs -c -k all
update-initramfs: Generating /boot/initrd.img-6.1.0-18-amd64
update-initramfs: Generating /boot/initrd.img-bootmenu
W: zstd compression (CONFIG_RD_ZSTD) not supported by kernel, using gzip
E: gzip compression (CONFIG_RD_GZIP) not supported by kernel
update-initramfs: failed for /boot/initrd.img-bootmenu with 1.
root@debian:/# 
```

I added `CONFIG_RD_ZSTD=y` to `/boot/config-bootmenu` and got the following result:

```text
root@debian:/# update-initramfs -c -k all
update-initramfs: Generating /boot/initrd.img-6.1.0-18-amd64
update-initramfs: Generating /boot/initrd.img-bootmenu
W: missing /lib/modules/bootmenu
W: Ensure all necessary drivers are built into the linux image!
depmod: ERROR: Bad version passed bootmenu
cat: /var/tmp/mkinitramfs_6RYan1/lib/modules/bootmenu/modules.builtin: No such file or directory
W: Can't find modules.builtin.modinfo (for locating built-in drivers' firmware, supported in Linux >=5.2)
depmod: ERROR: Bad version passed bootmenu
root@debian:/# 
```

When I asked about this at <https://web.libera.chat/#zfsbootmenu> I was told it was not necessary for `update-initramfs` to see the ZFS modules since the initrd was already build.

At this point I crossed my fingers, held my breath(*) and rebooted. The system boot looped without apparentloy finding anything to boot nor asking for a bootable drive.

(*) Didn't really hold my breath. This is an ancient Supermicro server motherboiard that takes too long to post.
