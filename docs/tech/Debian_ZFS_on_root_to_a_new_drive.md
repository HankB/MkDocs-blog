# Debian Root on ZFS to a new drive

**First try** cleaned up version will follow after a second run through.

## Motivation

Old drives on a file server are old and decrepit and not long for this world. 

## Constraints

* Old motherboard does not support UEFI boot.
* Current install is on 500GB mirrored HDDs and using about 7% of capacity. The desire is to migrate to smaller SSDs. (240GB/120GB HDDs) The current installation is using about 7% of the HDDs.
* Current install uses the `bpool`/`rpool` split as specified in the instructions at <https://openzfs.github.io/openzfs-docs/Getting%20Started/Debian/Debian%20Bullseye%20Root%20on%20ZFS.html#>

## Plan

1. Format and partition the target SSD as described in the instructions above.
1. Create root abd boot pools as described. Option: Name `rpool` and `bpool` or provide new names to disambiguate? Cache files etc. will have the original names. Cannot create `bpool` with another `bpool` already imported.
1. `zfs send | zfs recv` the two filesystems from the existing install to the target SSD.
1. Perform necessary rescue operations to make the target SSD bootable.

These operations are being performed on a test system to minimize possible disruption with the "real" server until a working procedure is identified.

## Execution (2023-03-08)

Starting with a new unformatted SSD or one which has been secure erased. (`/dev/sdh`)

### Identify and secure erase

```text
root@acorn3:~# ls -l /dev/disk/by-id/wwn*|grep sdh
lrwxrwxrwx 1 root root  9 Mar  8 16:28 /dev/disk/by-id/wwn-0x5000c5002ff22a4c -> ../../sdh
lrwxrwxrwx 1 root root 10 Mar  8 16:28 /dev/disk/by-id/wwn-0x5000c5002ff22a4c-part1 -> ../../sdh1
lrwxrwxrwx 1 root root 10 Mar  8 16:28 /dev/disk/by-id/wwn-0x5000c5002ff22a4c-part2 -> ../../sdh2
lrwxrwxrwx 1 root root 10 Mar  8 16:28 /dev/disk/by-id/wwn-0x5000c5002ff22a4c-part3 -> ../../sdh3
lrwxrwxrwx 1 root root 10 Mar  8 16:28 /dev/disk/by-id/wwn-0x5000c5002ff22a4c-part4 -> ../../sdh4
root@acorn3:~# DISK=/dev/disk/by-id/wwn-0x5000c5002ff22a4c
root@acorn3:~# hdparm --security-set-pass NULL $disk
missing PASSWD
root@acorn3:~# hdparm --security-set-pass NULL $DISK
security_password: ""

/dev/disk/by-id/wwn-0x5000c5002ff22a4c:
 Issuing SECURITY_SET_PASS command, password="", user=user, mode=high
root@acorn3:~# hdparm --security-erase-enhanced NULL  $DISK
security_password: ""

/dev/disk/by-id/wwn-0x5000c5002ff22a4c:
 Issuing SECURITY_ERASE command, password="", user=user
root@acorn3:~# partprobe
root@acorn3:~# ls -l /dev/disk/by-id/wwn*|grep sdh
lrwxrwxrwx 1 root root  9 Mar  8 17:28 /dev/disk/by-id/wwn-0x5000c5002ff22a4c -> ../../sdh
root@acorn3:~# 
```

### Partition

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
    bpool_new ${DISK}-part3
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
    rpool_new ${DISK}-part4
```

Result

```text
root@acorn3:~# zpool list
NAME        SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
bpool       960M   218M   742M        -         -     4%    22%  1.00x    ONLINE  -
bpool_new   960M   612K   959M        -         -     0%     0%  1.00x    ONLINE  /mnt
rpool        28G  16.5G  11.5G        -         -    34%    58%  1.00x    ONLINE  -
rpool_new   222G   488K   222G        -         -     0%     0%  1.00x    ONLINE  /mnt
tank       1.81T   688G  1.14T        -         -     1%    37%  1.00x    ONLINE  -
root@acorn3:~# 
```

### send/recv pools

send options  
  `-R`  replicate  

recv options  
  `-F`  Force a rollback of the file system ... (required because `bpool_new` exists)  
  `-d`  Discard the first element of the sent snapshot's file system name  ...  
  `-u`  File system that is associated with the received stream is not mounted.  

```text
zfs snap -r bpool@migrate.0
zfs send -R bpool@migrate.0  | mbuffer | zfs recv -F -d -u bpool_new

zfs snap -r rpool@migrate.0 
zfs send -R rpool@migrate.0  | mbuffer | zfs recv -F -d -u rpool_new
```

```text
root@acorn3:~# zfs snap -r bpool@migrate.0
root@acorn3:~# zfs send -R bpool@migrate.0  | mbuffer | zfs recv -F -d -u bpool_new
in @ 52.0 MiB/s, out @ 52.0 MiB/s,  226 MiB total, buffer   0% full
summary:  227 MiByte in  3.4sec - average of 66.1 MiB/s
root@acorn3:~# zfs snap -r rpool@migrate.0 
root@acorn3:~# zfs send -R rpool@migrate.0  | mbuffer | zfs recv -F -d -u rpool_new
in @  0.0 kiB/s, out @  0.0 kiB/s, 24.1 GiB total, buffer   0% full
summary: 24.1 GiByte in 11min 28.6sec - average of 35.8 MiB/s
root@acorn3:~# 
```

Resulting filesystems (and comparison with sourde pools.)

```text
root@acorn3:~# zfs list -r bpool_new
NAME                    USED  AVAIL     REFER  MOUNTPOINT
bpool_new               217M   615M       96K  /mnt/boot
bpool_new/BOOT          216M   615M       96K  none
bpool_new/BOOT/debian   216M   615M      129M  /mnt/boot
root@acorn3:~# zfs list -r bpool
NAME                USED  AVAIL     REFER  MOUNTPOINT
bpool               218M   614M       96K  /boot
bpool/BOOT          216M   614M       96K  none
bpool/BOOT/debian   216M   614M      129M  /boot
root@acorn3:~# 
root@acorn3:~# 
root@acorn3:~# zfs list -r rpool_new -d2
NAME                    USED  AVAIL     REFER  MOUNTPOINT
rpool_new              16.4G   199G       96K  /mnt
rpool_new/ROOT         3.20G   199G       96K  none
rpool_new/ROOT/debian  3.20G   199G     2.24G  /mnt
rpool_new/home         11.3G   199G       96K  /mnt/home
rpool_new/home/hbarta  11.3G   199G     10.9G  /mnt/home/hbarta
rpool_new/home/root     764K   199G      264K  /mnt/root
rpool_new/srv            96K   199G       96K  /mnt/srv
rpool_new/usr          14.9M   199G       96K  /mnt/usr
rpool_new/usr/local    14.8M   199G     14.4M  /mnt/usr/local
rpool_new/var          1.83G   199G       96K  /mnt/var
rpool_new/var/cache     455M   199G      355M  /mnt/var/cache
rpool_new/var/games     152K   199G       96K  /mnt/var/games
rpool_new/var/lib      1.26G   199G       96K  /mnt/var/lib
rpool_new/var/log       125M   199G     96.7M  /mnt/var/log
rpool_new/var/mail      152K   199G       96K  /mnt/var/mail
rpool_new/var/snap      152K   199G       96K  /mnt/var/snap
rpool_new/var/spool    1.23M   199G      392K  /mnt/var/spool
rpool_new/var/tmp       592K   199G      128K  /mnt/var/tmp
rpool_new/var/www       228K   199G      108K  /mnt/var/www
root@acorn3:~# zfs list -r rpool -d2
NAME                USED  AVAIL     REFER  MOUNTPOINT
rpool              16.5G  10.7G       96K  /
rpool/ROOT         3.20G  10.7G       96K  none
rpool/ROOT/debian  3.20G  10.7G     2.24G  /
rpool/home         11.3G  10.7G       96K  /home
rpool/home/hbarta  11.3G  10.7G     10.9G  /home/hbarta
rpool/home/root     764K  10.7G      264K  /root
rpool/srv            96K  10.7G       96K  /srv
rpool/usr          14.9M  10.7G       96K  /usr
rpool/usr/local    14.8M  10.7G     14.4M  /usr/local
rpool/var          1.84G  10.7G       96K  /var
rpool/var/cache     455M  10.7G      355M  /var/cache
rpool/var/games     152K  10.7G       96K  /var/games
rpool/var/lib      1.27G  10.7G       96K  /var/lib
rpool/var/log       130M  10.7G     96.7M  /var/log
rpool/var/mail      152K  10.7G       96K  /var/mail
rpool/var/snap      152K  10.7G       96K  /var/snap
rpool/var/spool    1.36M  10.7G      392K  /var/spool
rpool/var/tmp       592K  10.7G      128K  /var/tmp
rpool/var/www       228K  10.7G      108K  /var/www
root@acorn3:~#
```

### Rescue

Performed using a Debian 11.6 live USB. (Question: Can this be done using the original install?) Exception: no need to install `debbootstrap` package. Import new pools:

```text
zpool import -f -N -R /mnt rpool_new
zpool import -f -N -R /mnt bpool_new
zfs mount rpool_new/ROOT/debian
```

Result

```text
root@debian:~# df /mnt
Filesystem            1K-blocks    Used Available Use% Mounted on
rpool_new/ROOT/debian 210641024 2346368 208294656   2% /mnt
root@debian:~# 
root@debian:~# zfs mount -a
root@debian:~# df|grep mnt
rpool_new/ROOT/debian             210641024  2346368 208294656   2% /mnt
bpool_new/BOOT/debian                761600   131712    629888  18% /mnt/boot
rpool_new/var/cache               208658432   363776 208294656   1% /mnt/var/cache
rpool_new/var/spool               208295168      512 208294656   1% /mnt/var/spool
rpool_new/usr/local               208309504    14848 208294656   1% /mnt/usr/local
rpool_new/var/snap                208294784      128 208294656   1% /mnt/var/snap
rpool_new/var/log                 208393728    99072 208294656   1% /mnt/var/log
rpool_new/var/tmp                 208294784      128 208294656   1% /mnt/var/tmp
rpool_new/home                    208294784      128 208294656   1% /mnt/home
rpool_new/var/mail                208294784      128 208294656   1% /mnt/var/mail
rpool_new/var/www                 208294784      128 208294656   1% /mnt/var/www
rpool_new/srv                     208294784      128 208294656   1% /mnt/srv
rpool_new/var/games               208294784      128 208294656   1% /mnt/var/games
rpool_new/home/root               208295040      384 208294656   1% /mnt/root
rpool_new/var/lib/nfs             208294784      128 208294656   1% /mnt/var/lib/nfs
rpool_new/var/lib/AccountsService 208294784      128 208294656   1% /mnt/var/lib/AccountsService
rpool_new/var/lib/docker          208329856    35200 208294656   1% /mnt/var/lib/docker
rpool_new/var/lib/NetworkManager  208294784      128 208294656   1% /mnt/var/lib/NetworkManager
rpool_new/home/hbarta             219751424 11456768 208294656   6% /mnt/home/hbarta
rpool_new/home/hbarta/Programming 208296960     2304 208294656   1% /mnt/home/hbarta/Programming
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

Question: Does the lack of an EFI partition and corresponding mount matter for a system that does not support EFI?

`mount -a` seems to have no effect as all filesystems are already mounted.

```text
root@debian:/# df
Filesystem                        1K-blocks     Used Available Use% Mounted on
rpool_new/ROOT/debian             210640768  2346368 208294400   2% /
bpool_new/BOOT/debian                761600   131712    629888  18% /boot
rpool_new/var/cache               208658176   363776 208294400   1% /var/cache
rpool_new/var/spool               208294912      512 208294400   1% /var/spool
rpool_new/usr/local               208309248    14848 208294400   1% /usr/local
rpool_new/var/snap                208294528      128 208294400   1% /var/snap
rpool_new/var/log                 208393472    99072 208294400   1% /var/log
rpool_new/var/tmp                 208294528      128 208294400   1% /var/tmp
rpool_new/home                    208294528      128 208294400   1% /home
rpool_new/var/mail                208294528      128 208294400   1% /var/mail
rpool_new/var/www                 208294528      128 208294400   1% /var/www
rpool_new/srv                     208294528      128 208294400   1% /srv
rpool_new/var/games               208294528      128 208294400   1% /var/games
rpool_new/home/root               208294784      384 208294400   1% /root
rpool_new/var/lib/nfs             208294528      128 208294400   1% /var/lib/nfs
rpool_new/var/lib/AccountsService 208294528      128 208294400   1% /var/lib/AccountsService
rpool_new/var/lib/docker          208329600    35200 208294400   1% /var/lib/docker
rpool_new/var/lib/NetworkManager  208294528      128 208294400   1% /var/lib/NetworkManager
rpool_new/home/hbarta             219751168 11456768 208294400   6% /home/hbarta
rpool_new/home/hbarta/Programming 208296704     2304 208294400   1% /home/hbarta/Programming
udev                                8150136        0   8150136   0% /dev
tmpfs                               8196588        0   8196588   0% /dev/shm
tmpfs                               8196588        0   8196588   0% /run
root@debian:/#
```

Perform sensible steps from <https://openzfs.github.io/openzfs-docs/Getting%20Started/Debian/Debian%20Bullseye%20Root%20on%20ZFS.html#step-4-system-configuration>

* `grub` is already installed so `dpkg-reconfigure grub-pc`. After identifying the new SSD as `/dev/sdi` choose that to install grub.
* `zfs-initramfs` is already installed and `dpkg-reconfigure zfs-initramfs` seems to do nothing.
* `/etc/systemd/system/zfs-import-bpool.service` but cannot check status of the service from within the chroot.
* `grub-probe /boot` reports `zfs`.
* `update-initramfs -c -k all` generates entries for three kernels w/out reporting any problems.
* `update-grub` finds the three initrd images and complains about `/dev/sdh2` (which is the USB holding the live image.)
* `grub-install /dev/sdi` completes without error.
* Cache files for `rpool` and `bpool` exist. Touched cache files for `rpool_new` and `bpool_new` and after execiuting the `zfs set` commands they are populated. Removed the cache files for `rpool` and `bpool`.
* Fixed (and confirmed) the mountpoints in the cache files.
* Exit the chroot.

* Unmount all `/mnt/...` mounts.
* `zpool export -a` exports `bpool_new` but not `rpool_new` which is apparently a known issue.
* Remove live boot media and reboot, selecting the new SSD in the BIOS settings.

No joy. Booting from new SSD, still imports and uses `rpool` (not `rpool_new`) Disconnected original drive and the SDD install still tries to import `rpool` and fails miserably. It seems likely that the initramfs still points to rpool and that should be updated after the pool names and mount points are fixed.

### Back in the rescue chroot

The filesystems did not mount as before except for the root filesystem.

```text
Filesystem            1K-blocks    Used Available Use% Mounted on
rpool_new/ROOT/debian 210627584 2346368 208281216   2% /
udev                    8150136       0   8150136   0% /dev
tmpfs                   8196588       0   8196588   0% /dev/shm
tmpfs                   8196588       8   8196580   1% /run
root@debian:/etc/zfs# 
```

The file `/etc/zfs/zpool.cache` references `bpool_new` but not `rpool_new`. Examination (using `vi`) showed it still used `rpool`. That was fixed by running `zed -F` in the backround and executing `zpool set cachefile=/etc/zfs/zpool.cache rpool_new`.

It looks like grub cannot update. Perhaps the fix for `zpool.cache` is sufficient. It is not.

## 2023-03-10 another try

Perform the entire process using a USB live so pools named `rpool` and `bpool` can be created to sidestep the renaming issue. The taret drive is identified as `/dev/sdi` and full ID is 

```text
DISK=/dev/disk/by-id/wwn-0x5000c5002ff22a4c
```

Secure erase followed by partition

```text
root@debian:~# DISK=/dev/disk/by-id/wwn-0x5000c5002ff22a4c
root@debian:~# ls -l $DISK
lrwxrwxrwx 1 root root 9 Mar 10 16:15 /dev/disk/by-id/wwn-0x5000c5002ff22a4c -> ../../sdi
root@debian:~#  hdparm --security-set-pass NULL  $DISK
security_password: ""

/dev/disk/by-id/wwn-0x5000c5002ff22a4c:
 Issuing SECURITY_SET_PASS command, password="", user=user, mode=high
root@debian:~# hdparm --security-erase-enhanced NULL $DISK
security_password: ""

/dev/disk/by-id/wwn-0x5000c5002ff22a4c:
 Issuing SECURITY_ERASE command, password="", user=user
root@debian:~# sgdisk -a1 -n1:24K:+1000K -t1:EF02 $DISK
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
root@debian:~# sgdisk     -n2:1M:+512M   -t2:EF00 $DISK
The operation has completed successfully.
root@debian:~# sgdisk     -n3:0:+1G      -t3:BF01 $DISK
The operation has completed successfully.
root@debian:~# sgdisk     -n4:0:0        -t4:BF00 $DISK
The operation has completed successfully.
root@debian:~# sgdisk -p $DISK
Disk /dev/disk/by-id/wwn-0x5000c5002ff22a4c: 468862128 sectors, 223.6 GiB
Sector size (logical/physical): 512/4096 bytes
Disk identifier (GUID): 4D6A0CF9-23E5-49B1-BF0A-47865F401CD9
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
root@debian:~# 
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

Old pools are imported using their `id` and given new names. (This will probably make the old install unbootable until the names are changed back.)

```text
root@debian:~# zpool import
   pool: testz
     id: 16927082409323128445
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

        testz                                          ONLINE
          raidz1-0                                     ONLINE
            ata-WDC_WD2003FYPS-27Y2B0_WD-WCAVY5836470  ONLINE
            ata-WDC_WD2003FYPS-27Y2B0_WD-WCAVY7294152  ONLINE
            ata-WDC_WD2003FYPS-27Y2B0_WD-WCAVY6148882  ONLINE

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

   pool: tank
     id: 7362029663796406372
  state: ONLINE
status: The pool was last accessed by another system.
 action: The pool can be imported using its name or numeric identifier and
        the '-f' flag.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-EY
 config:

        tank                                           ONLINE
          mirror-0                                     ONLINE
            ata-WDC_WD2002FYPS-02W3B0_WD-WCAVY7272361  ONLINE
            ata-WDC_WD2003FYPS-27Y2B0_WD-WCAVY6466521  ONLINE

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
root@debian:~# zpool import -f 2081542946275218239
cannot import 'bpool': pool already exists
root@debian:~# zpool import -f 2081542946275218239 bpool_old
root@debian:~# zpool import -f 4536037978216782426 rpool_old
root@debian:~# 
```

Commands to send/recv pools

```text
zfs snap -r bpool_old@migrate.1
zfs send -R bpool_old@migrate.1  | mbuffer | zfs recv -F -d -u bpool

zfs snap -r rpool_old@migrate.1 
zfs send -R rpool_old@migrate.1  | mbuffer | zfs recv -F -d -u rpool
```

```text
root@debian:~# zfs snap -r bpool_old@migrate.1
root@debian:~# zfs send -R bpool_old@migrate.1  | mbuffer | zfs recv -F -d -u bpool
in @ 39.9 MiB/s, out @ 43.9 MiB/s,  200 MiB total, buffer   0% full
summary:  227 MiByte in  3.5sec - average of 64.9 MiB/s
root@debian:~# zfs snap -r rpool_old@migrate.1 
root@debian:~# zfs send -R rpool_old@migrate.1  | mbuffer | zfs recv -F -d -u rpoolmount --make-private --rbind /dev  /mnt_new/dev
in @  0.0 kiB/s, out @  0.0 kiB/s, 24.2 GiB total, buffer   0% full
summary: 24.3 GiByte in 10min 50.9sec - average of 38.2 MiB/s
root@debian:~# 
```

Next start rescue process.

```text
root@debian:~# zpool export -a
root@debian:~# zpool import -N -R /mnt rpool
root@debian:~# zpool import -N -R /mnt bpool
root@debian:~# zfs mount rpool/ROOT/debian
root@debian:~# 
root@debian:~# mount --make-private --rbind /dev  /mnt/dev
root@debian:~# zfs mount -a
root@debian:~# zfs list
```

Update initramfs and update grtub and install grub to SSD and reboot. On first reboot `rpool` from the HDD was imported (despite being renamed). Following shutdown and disconnection of the HDD, the system booted from the SSD and following manual import of the new `rpool` it came up normally. Yay!
