# Raspberry Pi installed with root on ZFS

Following a blend of the instrustions at <https://openzfs.github.io/openzfs-docs/Getting%20Started/Ubuntu/Ubuntu%2022.04%20Root%20on%20ZFS%20for%20Raspberry%20Pi.html#> and <https://openzfs.github.io/openzfs-docs/Getting%20Started/Debian/Debian%20Bullseye%20Root%20on%20ZFS.html> and the information about configuring Debian on Raspberry Pi at <https://raspi.debian.net/defaults-and-settings/>

## Motivation

Debian on a Raspberry P (4B, CM4.)

## Step by step

Step numbers refer to the Ubuntu instructions. Initial provisioning is performed on an X86_64 desktop running Debian Bookworm. Install target is a SATA SSD which will be connected to a Pi 4B. The instructions are for Bullseye but Bookworm (Pi) images will be used. Encryption will not be explored at this time.

## Step 1: Disk Formatting

### 1. Download and unpack the official image:

```text
curl https://gemmei.ftp.acc.umu.se/cdimage/unofficial/raspi//daily/raspi_4_bookworm.img.xz | xzcat >$(date +%Y%m%d)_raspi_4_bookworm.img
```

### 2. Dump the partition table for the image:

```text
sfdisk -d 20230901_raspi_4_bookworm.img
```

resulting in

```text
hbarta@olive:~/Downloads/Debian_pi_root_ZFS$ sfdisk -d 20230901_raspi_4_bookworm.img 
label: dos
label-id: 0x91e11d01
device: 20230901_raspi_4_bookworm.img
unit: sectors
sector-size: 512

20230901_raspi_4_bookworm.img1 : start=        8192, size=     1040384, type=c
20230901_raspi_4_bookworm.img2 : start=     1048576, size=     4071424, type=83
hbarta@olive:~/Downloads/Debian_pi_root_ZFS$
```

The required variables are then 

```text
BOOT=1040384
ROOT=4071424
```

### 3. Create a partition script:

```text
cat > partitions << EOF
label: dos
unit: sectors

1 : start=  2048,  size=$BOOT,  type=c, bootable
2 : start=$((2048+BOOT)),  size=$ROOT, type=83
3 : start=$((2048+BOOT+ROOT)), size=$ROOT, type=83
EOF
```

### 4. Connect the disk:

In this case `/dev/sdb`, and create the necessary variables.

```text
DISK=/dev/sdb
export DISKP=${DISK}
```

### 5. Ensure swap partitions are not in use:

```text
swapon -v
```

(None in use.)

### 6. Clear old ZFS labels:

```text
sudo zpool labelclear -f ${DISK}
```

(Secure erase or `blkdiscard` are alternate options for an SSD drive)

```text
sudo blkdiscard -f $DISK
```

### 7. Delete existing partitions:

```text
echo "label: dos" | sudo sfdisk ${DISK}
sudo partprobe
ls ${DISKP}*
```

(Not needed when `blkdiscard` is used)

### 8. Create the partitions:

```text
sudo sfdisk $DISK < partitions
```

### 9. Loopback mount the image:

```text
IMG=$(sudo losetup -fP --show 20230901_raspi_4_bookworm.img)
```

### 10. Copy the bootloader data:

```text
sudo dd if=${IMG}p1 of=${DISKP}1 bs=1M
```

### 11. Clear old label(s) from partition 2:

```text
sudo wipefs -a ${DISKP}2
```

### 12. Copy the root filesystem data:

```text
# NOTE: the destination is p3, not p2.
sudo dd if=${IMG}p2 of=${DISKP}3 bs=1M status=progress conv=fsync
```

### 13. Unmount the image:

```text
sudo losetup -d $IMG
```

### 14. If setting up a USB disk:

This is where the Debian seems to differ from Ubuntu. `zcat -qf` on the installed image produces output that is bit for bit identical with the input. For now this will be left as is.

Debian provides configuration options that will be set here. In particular `root_authorized_key` is necessary in order to SSH in later since Debian does not create a default user account. It is also convenient to set `hostname` at this time.

```text
sudo -sE

MNT=$(mktemp -d /mnt/XXXXXXXX)
mkdir -p $MNT/boot
mount ${DISKP}1 $MNT/boot
vi $MNT/boot/sysconf.txt # add public key as root_authorized_key
umount $MNT/*
rm -rf $MNT
exit
```

### 15. Boot the Raspberry Pi.

(Successful.)

## Step 2: Setup ZFS

### 1. Become root:

Determine IP address of the Pi SSH in as root.

### 2. Set the DISK and DISKP variables again:

```text
DISK=/dev/sda
DISKP=${DISK}
```

### 3. Install ZFS:

Instructions differ with Ubuntu as ZFS modules must be built for Debian.

```text
vi /etc/apt/sources.list # add "contrib" repo
apt update
apt install pv zfs-initramfs console-setup locales
dpkg-reconfigure locales
zfs --version
```

```text
root@mercury:~# zfs --version
zfs-2.1.11-1
zfs-kmod-2.1.11-1
root@mercury:~# 
```

### 4. Create the root pool:

(unencrypted)

```text
zpool create \
    -o ashift=12 \
    -O acltype=posixacl -O canmount=off -O compression=lz4 \
    -O dnodesize=auto -O normalization=formD -O relatime=on \
    -O xattr=sa -O mountpoint=/ -R /mnt \
    rpool ${DISKP}2
```

```text
root@mercury:~# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
rpool  1.88G   476K  1.87G        -         -     0%     0%  1.00x    ONLINE  /mnt
root@mercury:~# 
```

## Step 3: System Installation

### 1. Create a filesystem dataset to act as a container:

```text
zfs create -o canmount=off -o mountpoint=none rpool/ROOT
```

### 2. Create a filesystem dataset for the root filesystem:

Question: Are the `zsys` relkated properties useful for Debian? Also is the unique (UUID) name desired? Following Debian policy on this.

```text
zfs create -o canmount=noauto -o mountpoint=/ rpool/ROOT/debian
zfs mount rpool/ROOT/debian
```

### 3. Create datasets:

More possible differences in the way Ubuntu dreates a dataset for `/root`. Again following Debian policy (And copying the commands from the Debian instruyction page.) And not bothering with the optional datasets at this time but wiull create the tmpfs fpr `/tmp`

```text
zfs create                     rpool/home
zfs create -o mountpoint=/root rpool/home/root
chmod 700 /mnt/root
zfs create -o canmount=off     rpool/var
zfs create -o canmount=off     rpool/var/lib
zfs create                     rpool/var/log
zfs create -o com.sun:auto-snapshot=false  rpool/tmp
chmod 1777 /mnt/tmp
```

### 4. (Debian) Mount a tmpfs at /run:

```text
mkdir /mnt/run
mount -t tmpfs tmpfs /mnt/run
mkdir /mnt/run/lock
```

### 4. (Ubuntu) Optional: Ignore synchronous requests:

Not done.

### 5. Copy the system into the ZFS filesystems:

```text
(cd /; tar -cf - --one-file-system --warning=no-file-ignored .) | \
    pv -p -bs $(du -sxm --apparent-size / | cut -f1)m | \
    (cd /mnt ; tar -x)
```

## Step 4: System Configuration

### Step 1. Configure the hostname:

(Set earlier but `/etc/hosts` requires the addition.)

```text
vi /mnt/etc/hosts
```

### 2. Stop `zed``:

Not apparently running

```text
root@mercury:~# systemctl status zed
○ zfs-zed.service - ZFS Event Daemon (zed)
     Loaded: loaded (/lib/systemd/system/zfs-zed.service; enabled; preset: enabled)
     Active: inactive (dead)
  Condition: start condition failed at Fri 2023-09-01 16:24:08 UTC; 30min ago
       Docs: man:zed(8)

Sep 01 16:24:08 mercury systemd[1]: zfs-zed.service - ZFS Event Daemon (zed) was skipped because of an unmet condition check (ConditionPathIsDire>
root@mercury:~# 
```

### 3. Bind the virtual filesystems from the running environment to the new ZFS environment and chroot into it:

```text
mount --make-private --rbind /boot/firmware /mnt/boot/firmware
mount --make-private --rbind /dev  /mnt/dev
mount --make-private --rbind /proc /mnt/proc
mount --make-private --rbind /run  /mnt/run
mount --make-private --rbind /sys  /mnt/sys
chroot /mnt /usr/bin/env DISK=$DISK UUID=$UUID bash --login
```

### 4. Configure a basic system environment:

```text
apt update
dpkg-reconfigure locales
dpkg-reconfigure tzdata
```

### 5. For LUKS installs only, setup `/etc/crypttab`:

Not needed.

### 6. Optional: Mount a tmpfs to /tmp

```text
cp /usr/share/systemd/tmp.mount /etc/systemd/system/
systemctl enable tmp.mount
```

### 7. Setup system groups:

```text
addgroup --system lpadmin
addgroup --system sambashare
```

### 8. Fix filesystem mount ordering:

```text
zfs set canmount=noauto rpool/ROOT/debian
cat /etc/zfs/zfs-list.cache/rpool 
fg
<ctrl>C
sed -Ei "s|/mnt/?|/|" /etc/zfs/zfs-list.cache/*
```

### 9. Remove old filesystem from `/etc/fstab`:

```text
vi /etc/fstab
# Remove the old root filesystem line:
#   LABEL=RASPIROOT / ext4 ...
```

### (Debian) update initramfs

```text
update-initramfs -c -b /boot/firmware -k all
```

### 10. Configure kernel command line:

```text
cp /boot/firmware/cmdline.txt /boot/firmware/cmdline.txt.bak
sed -i "s|root=LABEL=RASPIROOT|root=ZFS=rpool/ROOT/debian|" \
    /boot/firmware/cmdline.txt
sed -i "s| fixrtc||" /boot/firmware/cmdline.txt
sed -i "s|$| init_on_alloc=0|" /boot/firmware/cmdline.txt
```

### 11. Optional (but highly recommended): Make debugging booting easier:

```text
sed -i "s|$| nosplash|" /boot/firmware/cmdline.txt
```

```text
root@mercury:/# cat /boot/firmware/cmdline.txt
console=tty0 console=ttyS1,115200 root=ZFS=rpool/ROOT/debian rw fsck.repair=yes net.ifnames=0  rootwait  init_on_alloc=0 nosplash
root@mercury:/# 
```

12. (Cross fingers and) Reboot

```text
exit
reboot
```

System stops at 

```text
[    7.nnnnnn] sd 0:0:0:0: [sda] Attached SCSI
```

Maybe need to run `update-initramfs -c -b /boot/firmware -k all` during installation. Attempt to rescue - no joy. Try the full install again. No joy. Need to step back and figure out what's not working.
