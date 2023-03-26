# KDE Bookworm

Testing the KDE Plasma desktop on Debian Bookworm on a Pi 4B running the 64 bit Debian (not Raspberry Pi) OS.

## 2023-03-15 Previous

During testing to confirm that the bug <https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1019700> had been fixed, it was noted that the desktop hung when logging in. In order to test the bug, the system was upgraded to Debian Unstable. When the system was on Debian Testing, the same issue occurred. Since Testing is in hard freeze (and to be released as Bookworm) the hang warranted further investigation  and on Bookworm. This Pi 4B is ordinarily run on a different SSD with Bullseye installed and has not had this problem.

## 2023-03-15 Process

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

## Conclusion

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
