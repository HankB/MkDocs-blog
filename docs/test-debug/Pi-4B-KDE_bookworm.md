# KDE Plasma X11 login hang

Host: Debian Bookworm on Raspberry Pi 4B (USB/SSD boot)

Reported at <https://bugs.kde.org/show_bug.cgi?id=467812>

Testing the KDE Plasma desktop on Debian Bookworm on a Pi 4B running the 64 bit Debian (not Raspberry Pi) OS. Bookworm is a pending Debian release which at present (2023-03-25) is in "hard freeze" and it is important to get all functionality working.

## 2023-03-25 Previous

During testing to confirm that the bug <https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1019700> had been fixed, it was noted that the desktop hung when logging in. In order to test the bug, the system was upgraded to Debian Unstable. When the system was on Debian Testing, the same issue occurred. Since Testing is in hard freeze (and to be released as Bookworm) the hang warranted further investigation  and on Bookworm. This Pi 4B is ordinarily run on a different SSD with Bullseye installed and has not had this problem.

## 2023-03-25 Process

1. Image the SSD with  downloaded from <https://raspi.debian.net/tested-images/>.
1. Boot and allow to resize and reboot and login. (Rebooting as necessary to get a session that did not exhibit the emmc timeout issue.)
1. Housekeeping: hostname (`charm`), time zone, install `sudo` and `vim` and add user `hbarta`.
1. `apt update`, `apt upgrade`
1. Install a kernel (from Unstable) `linux-image-6.1.0-7-arm64_6.1.20-1_arm64.deb` downloaded from <https://packages.debian.org/sid/linux-image-6.1.0-7-arm64>. No additional dependencies were required.
1. Reboot.
1. Install a minimal KDE Plasma desktop (`apt install plasma-desktop`)
1. Reboot. No DM. No `xinit`. No `startx`. Went ahead and installed `sddm`.
1. Start `sddm` which brings up DM.
1. Login in. Works as expected. Logout.
1. Login using Wayland. Works as expected.

After concluding that everything was good, I continued testing and found myself unable to log in again. Additional operations included:

1. Install Chromium and Konsole.
1. Overclocking to 2MHz
1. Back out of the overclock.
1. Can't login.
1. Add another user `arnold` and try to login as the new user. Still hangs.

## ~~Conclusion~~ 2023-03-25

Seems like a non-problem. Will continue to test. ... Sadly after some usage, login once again hangs. Will repeat the test.

1. Image the SSD with  downloaded from <https://raspi.debian.net/tested-images/>.
1. Boot and allow to resize and reboot and login. (Rebooting as necessary to get a session that did not exhibit the emmc timeout issue.)
1. Housekeeping: hostname (`charm`), time zone and add user `hbarta`.
1. `apt update`, `apt upgrade`
1. Install a kernel (from Unstable) `linux-image-6.1.0-7-arm64_6.1.20-1_arm64.deb` downloaded from <https://packages.debian.org/sid/linux-image-6.1.0-7-arm64>. No additional dependencies were required.
1. `apt install plasma-desktop sddm konsole chromium vim -y`
1. Reboot.
1. Login (defaults to Wayland) and looks OK.
1. Logout and start monitors `dmesg -follow`, `cd /var/log;tail -f Xorg.0.log`
1. Login (Wayland) and all looks good.
1. Logout
1. Login (Wayland) and all looks good.
1. Start and exit Konsole
1. Start and exit Chromium
1. logout and login
1. Open Chromium. and open google.com. OK. Try to open second page and browser is foobar. Doesn't echo typing, finally able to open a second page (local MkDocs notes) and display is incomplete.
1. Open system monitor and Konsole and they work OK. Close all and logout.
1. Login to X.org session. Login hangs.
1. `systemctl restart sddm`
1. Login to X.org session. Login hangs.
1. `systemctl restart sddm`
1. Login to Wayland session. Login OK.

## 2023-03-26 Filing a bug report

1. Collect information
1. Prepare pastebin content

## 2023-03-26 try XFCE

Test two things:

1. X11 works with another DE
1. XFCE installs `lightdm`, can test that with KDE

Result: 

* XFCE (on X11) starts w/out issue using `lightdm` or `sddm`. 
* Plasma/X11 hangs when started using `lightdm`.
* Plasma/Wayland starts normally from `lightdm`.

Bug report at <https://bugs.kde.org/show_bug.cgi?id=467812>

## 2023-04-29 further patches to test.

Ref: <https://lore.kernel.org/all/20230428223500.23337-5-jim2101024@gmail.com/> and <https://lore.kernel.org/all/20230428223500.23337-1-jim2101024@gmail.com/>  
Suggested by kibi 

> fetch the patch series from the LKML, and apply it either on the base commit mentioned there, or to current mainline master  
>(b4 is super useful, `b4 mbox $message_id` and you got an mbox ready to `git am`)
