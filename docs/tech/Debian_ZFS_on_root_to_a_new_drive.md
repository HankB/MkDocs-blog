# Debian Root on ZFS to a new drive

## Motivation

Old drives on a file server are old and decrepit and not long for this world. 

## Contributing

This description is licensed under `Creative Commons Zero v1.0 Universal`. I appreciate contributions including:

* pointing out typos
* identifying descriptions that are ambiguous or otherwise not clear
* identifying obvious or not so obvious errors or ommisions
* describing better ways to do any or all of this
* anything else that improves these instructions

Particular areas that would benefit

* other ways to manage pool renaming
* usage for systems that employ EFI boot

Contributions can be submitted as issues or pull requests or direct contact. I will credit any contributors here.

* Hank Barta - original version
* Sam Van den Eynde - clone w/out live environment and get renaming right

The actual repo for this document is <https://github.com/HankB/MkDocs-blog>.

## Initial setup

DEBIAN 11 (Bullseye) with ZFS on root following the instructions at <https://openzfs.github.io/openzfs-docs/Getting%20Started/Debian/Debian%20Bullseye%20Root%20on%20ZFS.html#>

## Constraints

* Old motherboard does not support UEFI boot.
* Current (target) install is on 500GB mirrored HDDs and using about 7% of capacity. The desire is to migrate to smaller SSDs (240GB SSDs.)
* Current install uses the `bpool`/`rpool` split as specified in the instructions referenced above.

## Plan

1. Format and partition the target SSD as described in the instructions above.
1. Create root and boot pools as described and named to match the original pools: `rpool` and `bpool`.
1. `zfs send | zfs recv` the two filesystems from the existing install to the target SSD.
1. Perform necessary rescue operations to make the target SSD bootable.

These operations are being performed on a test system to minimize possible disruption with the "real" server until a working procedure is identified.

## Issues

Pool naming was a continuing problem. With new pools given different names, the cache files were wrong and the pools would not import at boot. This is probably a solvable problem, but the solution eluded me during several tries. The first successful attempt was to create the new pools in the live environment and name them to match the existing (old) pools. The old pools could then be imnported using different names, the related filesystems transferred and the rescue operations completed.

## Execution (2023-03-10)

Boot a Debian 11.6 live ISO (debian-live-11.6.0-amd64-gnome+nonfree.iso) using Ventoy. Install and start `openssh-server` and go into power settings to disable automatic suspend.

Add `contrib` to `/etc/apt/sources.list`, updat and install ZFS.

Identify the target SSD, in this case `/dev/sdi`

### Identify and secure erase

```text
root@debian:~# ls -l /dev/disk/by-id/wwn*|grep sdi
lrwxrwxrwx 1 root root  9 Mar 10 18:18 /dev/disk/by-id/wwn-0x5000c5002ff22a4c -> ../../sdi
lrwxrwxrwx 1 root root 10 Mar 10 18:18 /dev/disk/by-id/wwn-0x5000c5002ff22a4c-part1 -> ../../sdi1
lrwxrwxrwx 1 root root 10 Mar 10 18:18 /dev/disk/by-id/wwn-0x5000c5002ff22a4c-part2 -> ../../sdi2
lrwxrwxrwx 1 root root 10 Mar 10 18:18 /dev/disk/by-id/wwn-0x5000c5002ff22a4c-part3 -> ../../sdi3
lrwxrwxrwx 1 root root 10 Mar 10 18:18 /dev/disk/by-id/wwn-0x5000c5002ff22a4c-part4 -> ../../sdi4
root@debian:~# DISK=/dev/disk/by-id/wwn-0x5000c5002ff22a4c\
> 
root@debian:~# hdparm --security-set-pass NULL $DISK
security_password: ""

/dev/disk/by-id/wwn-0x5000c5002ff22a4c:
 Issuing SECURITY_SET_PASS command, password="", user=user, mode=high
root@debian:~# hdparm --security-erase-enhanced NULL  $DISK
security_password: ""

/dev/disk/by-id/wwn-0x5000c5002ff22a4c:
 Issuing SECURITY_ERASE command, password="", user=user
root@debian:~# partprobe
root@debian:~# ls -l /dev/disk/by-id/wwn*|grep sdi
lrwxrwxrwx 1 root root  9 Mar 10 19:56 /dev/disk/by-id/wwn-0x5000c5002ff22a4c -> ../../sdi
root@debian:~# 
```

### Partition

```text
sgdisk -a1 -n1:24K:+1000K -t1:EF02 $DISK
sgdisk     -n2:1M:+512M   -t2:EF00 $DISK
sgdisk     -n3:0:+1G      -t3:BF01 $DISK
sgdisk     -n4:0:0        -t4:BF00 $DISK
sgdisk -p $DISK
```

```text
root@acorn3:~# sgdisk -a1 -n1:24K:+1000K -t1:EF02 $DISK
Warning: Partition table header claims that the size of partition table
entries is 0 bytes, but this program  supports only 128-byte entries.
Adjusting accordingly, but partition table may be garbage.
Warning: Partition table header claims that the size of partition table
entries is 0 bytes, but this program  supports only 128-byte entries.
Adjusting accordingly, but partition table may be garbage.
Creating new GPT entries in memory.
Warning: Setting alignment to a value that does not match the disk's
physical block size! Performance degradation may result!
Physical block size = 4096
Logical block size = 512
Optimal alignment = 8 or multiples thereof.
The operation has completed successfully.
root@acorn3:~# sgdisk     -n2:1M:+512M   -t2:EF00 $DISK
The operation has completed successfully.
root@acorn3:~# sgdisk     -n3:0:+1G      -t3:BF01 $DISK
The operation has completed successfully.
root@acorn3:~# sgdisk     -n4:0:0        -t4:BF00 $DISK
The operation has completed successfully.
root@acorn3:~# sgdisk -p $DISK
Disk /dev/disk/by-id/wwn-0x5000c5002ff22a4c: 468862128 sectors, 223.6 GiB
Sector size (logical/physical): 512/4096 bytes
Disk identifier (GUID): ADDCB737-EF88-4B31-80B0-2F3881377244
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 468862094
Partitions will be aligned on 16-sector boundaries
Total free space is 14 sectors (7.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1              48            2047   1000.0 KiB  EF02  
   2            2048         1050623   512.0 MiB   EF00  
   3         1050624         3147775   1024.0 MiB  BF01  
   4         3147776       468862094   222.1 GiB   BF00  
root@acorn3:~# 
```

### Create pools

```text
zpool create \
    -o ashift=12 \
    -o autotrim=on -d \
    -o cachefile=/etc/zfs/zpool.cache \
    -o feature@async_destroy=enabled \
    -o feature@bookmarks=enabled \
    -o feature@embedded_data=enabled \
    -o feature@empty_bpobj=enabled \
    -o feature@enabled_txg=enabled \
    -o feature@extensible_dataset=enabled \
    -o feature@filesystem_limits=enabled \
    -o feature@hole_birth=enabled \
    -o feature@large_blocks=enabled \
    -o feature@livelist=enabled \
    -o feature@lz4_compress=enabled \
    -o feature@spacemap_histogram=enabled \
    -o feature@zpool_checkpoint=enabled \
    -O devices=off \
    -O acltype=posixacl -O xattr=sa \
    -O compression=lz4 \
    -O normalization=formD \
    -O relatime=on \
    -O canmount=off -O mountpoint=/boot -R /mnt \
    bpool ${DISK}-part3
```

```text
zpool create \
    -o ashift=12 \
    -o autotrim=on \
    -O acltype=posixacl -O xattr=sa -O dnodesize=auto \
    -O compression=lz4 \
    -O normalization=formD \
    -O relatime=on \
    -O canmount=off -O mountpoint=/ -R /mnt \
    rpool ${DISK}-part4
```

Result

```text
root@debian:~# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
bpool   960M   624K   959M        -         -     0%     0%  1.00x    ONLINE  /mnt
rpool   222G   420K   222G        -         -     0%     0%  1.00x    ONLINE  /mnt
root@debian:~# zfs list -r
NAME    USED  AVAIL     REFER  MOUNTPOINT
bpool   444K   831M       96K  /mnt/boot
rpool   420K   215G       96K  /mnt
root@debian:~# 
```

### Import source pools

Identify

```text
root@debian:~# zpool import
...
   pool: rpool
     id: 4536037978216782426
  state: ONLINE
status: The pool was last accessed by another system.
 action: The pool can be imported using its name or numeric identifier and
        the '-f' flag.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-EY
 config:

        rpool                                                      ONLINE
          ata-WDC_WD10S21X-24R1BT0-SSHD-8GB_WD-WXA1A74CP57R-part4  ONLINE

...
   pool: bpool
     id: 2081542946275218239
  state: ONLINE
status: The pool was last accessed by another system.
 action: The pool can be imported using its name or numeric identifier and
        the '-f' flag.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-EY
 config:

        bpool                                                      ONLINE
          ata-WDC_WD10S21X-24R1BT0-SSHD-8GB_WD-WXA1A74CP57R-part3  ONLINE
root@debian:~# 
```

```text
zpool import -f 4536037978216782426 rpool_old
zpool import -f 2081542946275218239 bpool_old
```

```text
root@debian:~# zpool import -f 4536037978216782426 rpool_old
root@debian:~# zpool import -f 2081542946275218239 bpool_old
root@debian:~# zpool list
NAME        SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
bpool       960M   624K   959M        -         -     0%     0%  1.00x    ONLINE  /mnt
bpool_old   960M   218M   742M        -         -     4%    22%  1.00x    ONLINE  -
rpool       222G   420K   222G        -         -     0%     0%  1.00x    ONLINE  /mnt
rpool_old    28G  16.5G  11.5G        -         -    34%    59%  1.00x    ONLINE  -
root@debian:~# 
```

### send/recv pools

send options  
  `-R`  replicate  

recv options  
  `-F`  Force a rollback of the file system ... (required because `bpool_new` exists)  
  `-d`  Discard the first element of the sent snapshot's file system name  ...  
  `-u`  File system that is associated with the received stream is not mounted.  

```text
zfs snap -r bpool_old@migrate.2
zfs send -R bpool_old@migrate.2  | mbuffer | zfs recv -F -d -u bpool

zfs snap -r rpool_old@migrate.2 
zfs send -R rpool_old@migrate.2  | mbuffer | zfs recv -F -d -u rpool
```

```text
root@debian:~# zfs snap -r bpool_old@migrate.2
root@debian:~# zfs send -R bpool_old@migrate.2  | mbuffer | zfs recv -F -d -u bpool
in @ 71.9 MiB/s, out @ 71.9 MiB/s,  216 MiB total, buffer   0% full
summary:  227 MiByte in  3.4sec - average of 67.8 MiB/s
root@debian:~# zfs snap -r rpool_old@migrate.2 
root@debian:~# zfs send -R rpool_old@migrate.2  | mbuffer | zfs recv -F -d -u rpool
in @  0.0 kiB/s, out @  0.0 kiB/s, 24.4 GiB total, buffer   0% fullll
summary: 24.4 GiByte in 10min 56.1sec - average of 38.1 MiB/s
root@debian:~# 
```

Resulting filesystems (and comparison with source pools.)

```text
root@debian:~# zfs list -r bpool
NAME                USED  AVAIL     REFER  MOUNTPOINT
bpool               217M   615M       96K  /mnt/boot
bpool/BOOT          216M   615M       96K  none
bpool/BOOT/debian   216M   615M      129M  /mnt/boot
root@debian:~# zfs list -r bpool_old
NAME                    USED  AVAIL     REFER  MOUNTPOINT
bpool_old               218M   614M       96K  /boot
bpool_old/BOOT          216M   614M       96K  none
bpool_old/BOOT/debian   216M   614M      129M  /boot
root@debian:~# 
root@debian:~# zfs list -r rpool -d2
NAME                USED  AVAIL     REFER  MOUNTPOINT
rpool              16.5G   199G       96K  /mnt
rpool/ROOT         3.21G   199G       96K  none
rpool/ROOT/debian  3.21G   199G     2.24G  /mnt
rpool/home         11.3G   199G       96K  /mnt/home
rpool/home/hbarta  11.3G   199G     10.9G  /mnt/home/hbarta
rpool/home/root     908K   199G      264K  /mnt/root
rpool/srv            96K   199G       96K  /mnt/srv
rpool/usr          15.0M   199G       96K  /mnt/usr
rpool/usr/local    14.9M   199G     14.4M  /mnt/usr/local
rpool/var          1.93G   199G       96K  /mnt/var
rpool/var/cache     517M   199G      355M  /mnt/var/cache
rpool/var/games     152K   199G       96K  /mnt/var/games
rpool/var/lib      1.28G   199G       96K  /mnt/var/lib
rpool/var/log       148M   199G     98.6M  /mnt/var/log
rpool/var/mail      152K   199G       96K  /mnt/var/mail
rpool/var/snap      152K   199G       96K  /mnt/var/snap
rpool/var/spool    1.75M   199G      436K  /mnt/var/spool
rpool/var/tmp       768K   199G      120K  /mnt/var/tmp
rpool/var/www       228K   199G      108K  /mnt/var/www
root@debian:~# zfs list -r rpool_old -d2
NAME                    USED  AVAIL     REFER  MOUNTPOINT
rpool_old              16.6G  10.6G       96K  /
rpool_old/ROOT         3.22G  10.6G       96K  none
rpool_old/ROOT/debian  3.21G  10.6G     2.24G  /
rpool_old/home         11.3G  10.6G       96K  /home
rpool_old/home/hbarta  11.3G  10.6G     10.9G  /home/hbarta
rpool_old/home/root     908K  10.6G      264K  /root
rpool_old/srv            96K  10.6G       96K  /srv
rpool_old/usr          15.0M  10.6G       96K  /usr
rpool_old/usr/local    14.9M  10.6G     14.4M  /usr/local
rpool_old/var          1.93G  10.6G       96K  /var
rpool_old/var/cache     517M  10.6G      355M  /var/cache
rpool_old/var/games     152K  10.6G       96K  /var/games
rpool_old/var/lib      1.28G  10.6G       96K  /var/lib
rpool_old/var/log       148M  10.6G     98.6M  /var/log
rpool_old/var/mail      152K  10.6G       96K  /var/mail
rpool_old/var/snap      152K  10.6G       96K  /var/snap
rpool_old/var/spool    1.96M  10.6G      436K  /var/spool
rpool_old/var/tmp       768K  10.6G      120K  /var/tmp
rpool_old/var/www       228K  10.6G      108K  /var/www
root@debian:~# 
```

### Rescue

`zpool import` shows the original pools still named `rpool_old` and `bpool_old` so there is no confusion importing the new pools by name.

```text
zpool export -a
zpool import -f -N -R /mnt rpool
zpool import -f -N -R /mnt bpool
zfs load-key -a # if any pools employ native encryption
zfs mount rpool/ROOT/debian
zfs mount -a
```

Result

```text
root@debian:~# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
bpool   960M   217M   743M        -         -     1%    22%  1.00x    ONLINE  /mnt
rpool   222G  16.5G   205G        -         -     0%     7%  1.00x    ONLINE  /mnt
root@debian:~# df -h
Filesystem                     Size  Used Avail Use% Mounted on
udev                           7.8G     0  7.8G   0% /dev
tmpfs                          1.6G  1.9M  1.6G   1% /run
/dev/mapper/ventoy             3.5G  3.5G     0 100% /run/live/medium
/dev/loop0                     2.9G  2.9G     0 100% /run/live/rootfs/filesystem.squashfs
tmpfs                          7.9G  306M  7.6G   4% /run/live/overlay
overlay                        7.9G  306M  7.6G   4% /
tmpfs                          7.9G     0  7.9G   0% /dev/shm
tmpfs                          5.0M     0  5.0M   0% /run/lock
tmpfs                          7.9G  8.0K  7.9G   1% /tmp
tmpfs                          1.6G  176K  1.6G   1% /run/user/1000
/dev/sdh3                       94G   20G   70G  22% /media/user/extra
rpool/ROOT/debian              201G  2.3G  199G   2% /mnt
bpool/BOOT/debian              744M  129M  616M  18% /mnt/boot
rpool/srv                      199G  128K  199G   1% /mnt/srv
rpool/var/cache                199G  355M  199G   1% /mnt/var/cache
rpool/home                     199G  128K  199G   1% /mnt/home
rpool/var/games                199G  128K  199G   1% /mnt/var/games
rpool/var/spool                199G  512K  199G   1% /mnt/var/spool
rpool/usr/local                199G   15M  199G   1% /mnt/usr/local
rpool/var/snap                 199G  128K  199G   1% /mnt/var/snap
rpool/home/root                199G  384K  199G   1% /mnt/root
rpool/var/log                  199G   99M  199G   1% /mnt/var/log
rpool/var/lib/docker           199G   35M  199G   1% /mnt/var/lib/docker
rpool/var/mail                 199G  128K  199G   1% /mnt/var/mail
rpool/var/lib/AccountsService  199G  128K  199G   1% /mnt/var/lib/AccountsService
rpool/var/lib/NetworkManager   199G  128K  199G   1% /mnt/var/lib/NetworkManager
rpool/var/tmp                  199G  128K  199G   1% /mnt/var/tmp
rpool/home/hbarta              210G   11G  199G   6% /mnt/home/hbarta
rpool/var/www                  199G  128K  199G   1% /mnt/var/www
rpool/var/lib/nfs              199G  128K  199G   1% /mnt/var/lib/nfs
rpool/home/hbarta/Programming  199G  2.3M  199G   1% /mnt/home/hbarta/Programming
root@debian:~# 
```

Chroot into new install

```text
root@debian:~# mount --make-private --rbind /dev  /mnt/dev
root@debian:~# mount --make-private --rbind /proc /mnt/proc
root@debian:~# mount --make-private --rbind /sys  /mnt/sys
root@debian:~# mount -t tmpfs tmpfs /mnt/run
root@debian:~# mkdir /mnt/run/lock
root@debian:~# chroot /mnt /bin/bash --login
root@debian:/# mount /boot
mount: /boot: can't find in /etc/fstab.
root@debian:/# cat /etc/fstab
# UNCONFIGURED FSTAB FOR BASE SYSTEM
root@debian:/# 
```

`mount -a` seems to have no effect as all filesystems are already mounted.

```text
root@debian:/# df| wc -l
24
root@debian:/# mount -a
root@debian:/# df| wc -l
24
root@debian:/# 
```

```text
root@debian:/# df -h
Filesystem                     Size  Used Avail Use% Mounted on
rpool/ROOT/debian              201G  2.3G  199G   2% /
bpool/BOOT/debian              744M  129M  616M  18% /boot
rpool/srv                      199G  128K  199G   1% /srv
rpool/var/cache                199G  355M  199G   1% /var/cache
rpool/home                     199G  128K  199G   1% /home
rpool/var/games                199G  128K  199G   1% /var/games
rpool/var/spool                199G  512K  199G   1% /var/spool
rpool/usr/local                199G   15M  199G   1% /usr/local
rpool/var/snap                 199G  128K  199G   1% /var/snap
rpool/home/root                199G  384K  199G   1% /root
rpool/var/log                  199G   99M  199G   1% /var/log
rpool/var/lib/docker           199G   35M  199G   1% /var/lib/docker
rpool/var/mail                 199G  128K  199G   1% /var/mail
rpool/var/lib/AccountsService  199G  128K  199G   1% /var/lib/AccountsService
rpool/var/lib/NetworkManager   199G  128K  199G   1% /var/lib/NetworkManager
rpool/var/tmp                  199G  128K  199G   1% /var/tmp
rpool/home/hbarta              210G   11G  199G   6% /home/hbarta
rpool/var/www                  199G  128K  199G   1% /var/www
rpool/var/lib/nfs              199G  128K  199G   1% /var/lib/nfs
rpool/home/hbarta/Programming  199G  2.3M  199G   1% /home/hbarta/Programming
udev                           7.8G     0  7.8G   0% /dev
tmpfs                          7.9G     0  7.9G   0% /dev/shm
tmpfs                          7.9G     0  7.9G   0% /run
root@debian:/# 
```

Perform sensible steps from <https://openzfs.github.io/openzfs-docs/Getting%20Started/Debian/Debian%20Bullseye%20Root%20on%20ZFS.html#step-4-system-configuration>

* Confirm that `/dev/sdi` is the boot device.
* `grub` is already installed.
* `zfs-initramfs` is already installed.
* `grub-probe /boot` reports `zfs`.
* `update-initramfs -c -k all` generates entries for three kernels w/out reporting any problems.
* `update-grub` finds the three initrd images and complains about `/dev/sdh2` (which is the USB holding the live image.)
* `grub-install /dev/sdi` completes without error.
* Cache files for `rpool` and `bpool` exist and have the correct mount points (`/etc/zfs/zfs-list.cache/bpool` ands `/etc/zfs/zfs-list.cache/rpool`).
* The cache binary cache file `/etc/zfs/zpool.cache` appears to have the correct pool name but the wrong drive name. This does not seem to cause any problem.
* Exit the chroot.
* Unmount all `/mnt/...` mounts.
* `zpool export -a` exports `bpool` but not `rpool` which is apparently a known issue.
* Power down.
* Remove live boot media, disconnect the original boot drive(s) and reboot, selecting the new SSD in the BIOS settings. (If the original boot drive is still attached, the system may import the original `rpool` instead of the new `rpool.`)
* On first boot from the new system, it may be necessary to `zpool import -f rpool` at the `(initramfs)` prompt.

## Alternatives

`zfs-discuss` list user Sam Van den Eynde has described a procedure that does not require booting a live environment and manages the pool renaming issue that I struggled with in their email <https://zfsonlinux.topicbox.com/groups/zfs-discuss/T536970860f9f34b4-Mb51bc8a885a9665639d79a13>

## Errata

Despite renaming the pools on the original media, with the new SSD disconnected the original system booted but did not import the boot pool (now listed as `bpool_old`.) It was manually imported and given the original name name (`bpool`) and was automatically imported on subsequent boots.

On the first try to boot the SSD with the original HDD still connected, the original root pool was imported as `rpool` and the new boot pool was imported as `bpool`. It seems likely that something in a cache file was pointing to the old root pool and that overrode the rename during the rescue operation.
