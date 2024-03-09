# Void Linux, MBR boot using ZFSBootMenu

Motivation: Familiarize using syslinux with ZFSBootMenu (ZBM)on a host that uses legacy (MBR) boot. Eventually syslinux/ZBM will be used to boot Debian Bookworm with ZFS on root.

* <https://docs.zfsbootmenu.org/en/v2.3.x/guides/void-linux/syslinux-mbr.html#>
* <https://github.com/leahneukirchen/hrmpf/releases>

## 2024-03-09 Download and install to Ventoy

From <https://github.com/leahneukirchen/hrmpf/releases> download `hrmpf-x86_64-6.5.12_1-20231124.iso` (No peers for the torrent.) Confirm `sha256sum`. Copy to Ventoy USB drive.

## 2024-03-09 boot hrmpf

Boot `hrmpf` and select first option to run. Comes up to console with SSH server available. Login with `anon`/`voidlinux` and passwordless `sudo`.

## 2024-03-09 Configure Live Environment

### Source `/etc/os-release`

```text
source /etc/os-release
export ID
echo $ID
```

```text
bash-5.2# source /etc/os-release
bash-5.2# export ID
bash-5.2# echo $ID
void
bash-5.2# 
```

### Generate `/etc/hostid`

```text
zgenhostid -f 0x00bab10c
```

### Define disk variables

Confirm (Kingston) SSD device node. `/dev/sda` it is!

```text
bash-5.2# sgdisk /dev/sda -p
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
bash-5.2# 
```

```text
export BOOT_DISK="/dev/sda"
export BOOT_PART="1"
export BOOT_DEVICE="${BOOT_DISK}${BOOT_PART}"

export POOL_DISK="/dev/sda"
export POOL_PART="2"
export POOL_DEVICE="${POOL_DISK}${POOL_PART}"
```

### Disk preparation

```text
blkdiscard -f $BOOT_DISK
cat <<EOF | sfdisk "${POOL_DISK}"
label: dos
start=1MiB, size=512MiB, type=83, bootable
start=513MiB, size=+, type=83
EOF
```

```text
bash-5.2# blkdiscard -f $BOOT_DISK
blkdiscard: Operation forced, data will be lost!
bash-5.2# 

bash-5.2# cat <<EOF | sfdisk "${POOL_DISK}"
label: dos
start=1MiB, size=512MiB, type=83, bootable
start=513MiB, size=+, type=83
EOF
Checking that no-one is using this disk right now ... OK

Disk /dev/sda: 111.79 GiB, 120034123776 bytes, 234441648 sectors
Disk model: KINGSTON SA400S3
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Script header accepted.
>>> Created a new DOS disklabel with disk identifier 0x14b74275.
/dev/sda1: Created a new partition 1 of type 'Linux' and of size 512 MiB.
/dev/sda2: Created a new partition 2 of type 'Linux' and of size 111.3 GiB.
/dev/sda3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0x14b74275

Device     Boot   Start       End   Sectors   Size Id Type
/dev/sda1  *       2048   1050623   1048576   512M 83 Linux
/dev/sda2       1050624 234441647 233391024 111.3G 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
bash-5.2# 
```

## 2024-03-09 ZFS pool creation


### Create the zpool

```text
zpool create -f -o ashift=12 \
 -O compression=lz4 \
 -O acltype=posixacl \
 -O xattr=sa \
 -O relatime=on \
 -o autotrim=on \
 -o compatibility=openzfs-2.1-linux \
 -m none zroot "$POOL_DEVICE"
 zpool list
 ```

 ```text
 bash-5.2# zpool create -f -o ashift=12 \
 -O compression=lz4 \
 -O acltype=posixacl \
 -O xattr=sa \
 -O relatime=on \
 -o autotrim=on \
 -o compatibility=openzfs-2.1-linux \
 -m none zroot "$POOL_DEVICE"
bash-5.2#  zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
zroot   111G   588K   111G        -         -     0%     0%  1.00x    ONLINE  -
bash-5.2# 
```

### Create initial file systems

```text
zfs create -o mountpoint=none zroot/ROOT
zfs create -o mountpoint=/ -o canmount=noauto zroot/ROOT/${ID}
zfs create -o mountpoint=/home zroot/home

zpool set bootfs=zroot/ROOT/${ID} zroot
```

```text
bash-5.2# zfs create -o mountpoint=none zroot/ROOT
bash-5.2# zfs create -o mountpoint=/ -o canmount=noauto zroot/ROOT/${ID}
bash-5.2# zfs create -o mountpoint=/home zroot/home
bash-5.2# zpool set bootfs=zroot/ROOT/${ID} zroot
bash-5.2# zfs list -r
NAME              USED  AVAIL  REFER  MOUNTPOINT
zroot             972K   108G    96K  none
zroot/ROOT        192K   108G    96K  none
zroot/ROOT/void    96K   108G    96K  /
zroot/home         96K   108G    96K  /home
bash-5.2# 
```

### Export, then re-import with a temporary mountpoint of /mnt

```text
zpool export zroot
zpool import -N -R /mnt zroot
zfs mount zroot/ROOT/${ID}
zfs mount zroot/home
```

```text
bash-5.2# zpool export zroot
bash-5.2# zpool import -N -R /mnt zroot
bash-5.2# zfs mount zroot/ROOT/${ID}
bash-5.2# zfs mount zroot/home
bash-5.2# zfs list -r
NAME              USED  AVAIL  REFER  MOUNTPOINT
zroot             996K   108G    96K  none
zroot/ROOT        192K   108G    96K  none
zroot/ROOT/void    96K   108G    96K  /mnt
zroot/home         96K   108G    96K  /mnt/home
bash-5.2# 
```

### Verify that everything is mounted correctly

```text
bash-5.2# mount | grep mnt
zroot/ROOT/void on /mnt type zfs (rw,relatime,xattr,posixacl,casesensitive)
zroot/home on /mnt/home type zfs (rw,relatime,xattr,posixacl,casesensitive)
bash-5.2# 
```

### Update device symlinks

```text
udevadm trigger
```

### Install Void

```text
XBPS_ARCH=x86_64 xbps-install \
  -S -R https://mirrors.servercentral.com/voidlinux/current \
  -r /mnt base-system
```

```text
...
base-system-0.114_1: configuring ...
base-system-0.114_1: installed successfully.

130 downloaded, 130 installed, 0 updated, 130 configured, 0 removed.
bash-5.2# 
```

### Copy our files into the new install

```text
cp /etc/hostid /mnt/etc
```

## 2024-03-09 Basic Void configuration

### Set the keymap, timezone and hardware clock

```text
cat << EOF >> /etc/rc.conf
KEYMAP="us"
HARDWARECLOCK="UTC"
EOF
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```

```text
[xchroot /mnt] # cat << EOF >> /etc/rc.conf
KEYMAP="us"
HARDWARECLOCK="UTC"
EOF
ln -sf /usr/share/zoneinfo/<timezone> /etc/localtime
bash: timezone: No such file or directory
[xchroot /mnt] # ls /usr/share/zoneinfo/America/Chicago
/usr/share/zoneinfo/America/Chicago
[xchroot /mnt] # ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
[xchroot /mnt] # 
```

### Configure your glibc locale

```text
cat << EOF >> /etc/default/libc-locales
en_US.UTF-8 UTF-8
en_US ISO-8859-1
EOF
xbps-reconfigure -f glibc-locales
```

```text
[xchroot /mnt] # cat << EOF >> /etc/default/libc-locales
en_US.UTF-8 UTF-8
en_US ISO-8859-1
EOF
xbps-reconfigure -f glibc-locales
glibc-locales: configuring ...
Generating GNU libc locales...
  en_US.UTF-8... done.
  en_US.ISO-8859-1... done.
glibc-locales: configured successfully.
[xchroot /mnt] # 
```

### Set a root password

```text
passwd
```

## 2024-03-09 ZFS Configuration

### Configure Dracut to load ZFS support

```text
cat << EOF > /etc/dracut.conf.d/zol.conf
nofsck="yes"
add_dracutmodules+=" zfs "
omit_dracutmodules+=" btrfs "
EOF
```

### Install ZFS

```text
xbps-install -S zfs
```

```text
[xchroot /mnt] # xbps-install -S zfs
[*] Updating repository `https://repo-default.voidlinux.org/current/x86_64-repodata' ...

Name                Action    Version           New version            Download size
bc                  install   -                 1.07.1_5               80KB 
binutils-doc        install   -                 2.41_2                 750KB 
binutils-libs       install   -                 2.41_2                 2188KB 
libunistring        install   -                 1.0_1                  583KB 
libidn2             install   -                 2.3.4_1                168KB 
public-suffix       install   -                 2024.03.01_1           128KB 
libpsl              install   -                 0.21.5_1               72KB 
libssh2             install   -                 1.11.0_2               121KB 
c-ares              install   -                 1.27.0_1               79KB 
jansson             install   -                 2.14_1                 29KB 
libev               install   -                 4.33_1                 31KB 
nghttp2             install   -                 1.59.0_1               711KB 
libcurl             install   -                 8.6.0_1                340KB 
libdebuginfod       install   -                 0.190_1                15KB 
binutils            install   -                 2.41_2                 4120KB 
kernel-libc-headers install   -                 6.1_1                  1328KB 
glibc-devel         install   -                 2.38_5                 3233KB 
libatomic           install   -                 13.2.0_2               11KB 
libatomic-devel     install   -                 13.2.0_2               11KB 
libgcc-devel        install   -                 13.2.0_2               1404KB 
libstdc++-devel     install   -                 13.2.0_2               2829KB 
gcc                 install   -                 13.2.0_2               71MB 
linux6.6-headers    install   -                 6.6.21_1               12MB 
linux-headers       install   -                 6.6_1                  542B 
make                install   -                 4.4.1_1                535KB 
dkms                install   -                 3.0.12_1               36KB 
libtirpc            install   -                 1.3.4_1                89KB 
libzfs              install   -                 2.2.3_1                1746KB 
gdbm                install   -                 1.23_1                 167KB 
libxcrypt-devel     install   -                 4.4.36_3               110KB 
perl                install   -                 5.38.2_3               14MB 
expat               install   -                 2.6.1_1                68KB 
libffi              install   -                 3.3_2                  19KB 
sqlite              install   -                 3.44.2_2               1245KB 
python3             install   -                 3.12.2_2               7785KB 
zfs                 install   -                 2.2.3_1                32MB 

Size to download:              159MB
Size required on disk:         515MB
Space available on disk:       106GB

Do you want to continue? [Y/n] 
...
