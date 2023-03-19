# Troubleshooting Debian root on ZFS

## Motivation

I'm sorry to hear that users are having difficulties with Debian root on ZFS. I've been running my laptop that way since I bought it in 2019 and have also converted my desktop to root on ZFS. More recently I've put my server on ZFS root. I can't say this has all been without problems, but I have been able to overcome any difficulties fairly easily. My goal with this document is to identify problems I've encountered and list the solutions I've employed.

These notes are focused on inability to boot. I use Debian so the notes are specific to that. Perhaps someone will submit a PR for other Linux distros.

## Contributions

Are welcome for anything from typos and formatting to better information and corrections or additions. Submit as issues, PRs or via direct contact. The repo for this is at <https://github.com/HankB/MkDocs-blog>

## TL;DR Rescue

Some times all that is needs is to import the pools from a prompt provided within the startup (e.g. `enter the root password or hit <ctrl>D to continue` I may not have the wording correct, but that is the gist of it.) Enter the root password and then manually import the root and boot pools. I ran into this recently when moving root and boot pools to another drive. It was the `(initramfs)` prompt and it was not necessary to enter the root password, just type `zpool import -f rpool` Hit `<ctrl>D` and the system comes up normally. I don't know why this happens, but the fix is easy. At other times the `zfs import` command will report that no modules are loaded and attempts to load them do not succeed. This could mean that the `initramfs` was not rebuilt or the modules themselves ere not rebuilt. Some kind of rescue is needed.

Debian leaves the previous kernel in the boot menu and the next thing I do is reboot and select the previous kernel. That should bring up the system with no difficulties. Then I try to rebuild the modules using

```text
dpkg-reconfigure zfs-dkms
```

Watch the output from this command to identify potential problems. If there are none, the system should then be able to boot the new kernel. If not, there may be other issues. Id so, read on.

## Can't load modules

This usually means that either the modules were not built for the kernel or the initramfs was not rebuilt for the newer kernel. Usually the command listed above fixes this or helps to identify the problem. Problems I have run into include.

### kernel headers not installed

If kernel headers are installed only for the specific (soon to be previous) kernel version, upgrading the kernel may not install the headers. On my Testing (currently Bookworm) laptop, I see the following headers listed.

```text
hbarta@rocinante:~/Documents/MkDocs-blog$ dpkg -l linux-headers*
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                         Version      Architecture Description
+++-============================-============-============-=========================================================
un  linux-headers                <none>       <none>       (no description available)
ii  linux-headers-6.1.0-5-amd64  6.1.12-1     amd64        Header files for Linux 6.1.0-5-amd64
ii  linux-headers-6.1.0-5-common 6.1.12-1     all          Common header files for Linux 6.1.0-5
ii  linux-headers-6.1.0-6-amd64  6.1.15-1     amd64        Header files for Linux 6.1.0-6-amd64
ii  linux-headers-6.1.0-6-common 6.1.15-1     all          Common header files for Linux 6.1.0-6
un  linux-headers-686-pae        <none>       <none>       (no description available)
ii  linux-headers-amd64          6.1.15-1     amd64        Header files for Linux amd64 configuration (meta-package)
un  linux-headers-generic        <none>       <none>       (no description available)
hbarta@rocinante:~/Documents/MkDocs-blog$ 
```

It is the `linux-headers-amd64` package that will pull in the appropriate headers for the next kernel.

### kernel modules not built

This can happen when the kernel ABI changes and the corresponding changes to the ZFS packages have not yet been released. Supported kernel versions for ZFS releases can be viewed at <https://github.com/openzfs/zfs/releases>. *Note:* That the modules may still build with an unsupported kernel version but there could be unexpected problems with operation.

In this situation it may be possible to try a release that has not yet been packaged for the OS or perhaps even try an unrelease version of ZFS. My personal preference is to wait for the corresponding ZFS package to released and packaged for a new kernel. (This is usually not encountered on Debian Stable as the kernel version is not updated unless one is using a backported kernel.)

Raspberry Pi OS does not (often? always?) rebuild the ZFS package, or at least the modules when a kernel is updated. The `dpkg-reconfigure zfs-dkms` command will fix this. Of course I have not found instructions for rooting Raspberry Pi OS on ZFS but these do exist for Ubuntu.
