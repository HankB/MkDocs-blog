# Testing networking (WiFi, ETH) Pi 4B

Testing the network interfaces on a Pi 4B running Debian Bookworm with available kernel packages. Following installation the `raspi-firmware1 package was held due to performance probnlems with the upgrade. All testing was performed with the firmware that came with the install media.

Initial tests were hampered by two issues.

* I had mistyped the WiFi password causing connection attempts to fail.
* Ethernet was unmanaged and remained configured even when physically disconnected. This masked WiFi even though WiFi appeared to be associated (wlan0 had an IP address handed out by the local DHCP server.)

I got the Ethernet under NetworkManager control and retyped the password and achieved some success with WiFi. Ethernet seemed to work without difficulty with all kernels.

## Status

I'm not happy with results, will retest when I have a chance (not till the weekend.)

## 5.15.0-2-arm64

This is the kernel that comes with the install media (`20220121_raspi_4_bookworm.img.xz`).

```text
Linux up 5.15.0-2-arm64 #1 SMP Debian 5.15.5-2 (2021-12-18) aarch64 GNU/Linux
```

No WiFi. `dmesg` capture at <https://paste.debian.net/1236231/> No `wlan0` in `ip addr` output.

Ethernet seems to work OK.

## 5.16.0-5-arm64

This is the kernel available in the repo to upgrade following installation.

```text
Linux up 5.16.0-5-arm64 #1 SMP Debian 5.16.14-1 (2022-03-15) aarch64 GNU/Linux
```

WiFi works w/out difficulty on both 5G and 2.4G APs. Switching repeatedly between the two SSIDs takes less than ten seconds.

## 5.17.0-rc8-arm64

Available from experimental

```text
Linux up 5.17.0-rc8-arm64 #1 SMP Debian 5.17~rc8-1~exp1 (2022-03-14) aarch64 GNU/Linux
```

WiFi works on 5G, does not associate on 2.4G. Captured output to `daemon.log` when trying to switch from an assopciated 5G AP to a 2.4G AP up until NM re-requested the WiFi password. <https://paste.debian.net/1236229/>

Connect times to the 5G AP was quick at times and not quick other times. At one point it was necessary to turn WiFi off and on again to get it to associate. (Both 5G and 2.4G rradios are on the same WiFi AP.)

Did not shutdown cleanly. Had to pull power.

## 5.17.0-trunk-arm64

From experimental

````text
Linux up 5.17.0-trunk-arm64 #1 SMP Debian 5.17.1-1~exp1 (2022-03-29) aarch64 GNU/Linux
```

Tested a second time and system hung. Hadn't remoted in - working from desktop. Happened once before and was unable to restart GDM and `shutdown` hung. Was able to examine logs and `dmesg` output and saw nothing unusual but may not have been looking in the right place. Had to remove power both times.

Both 5G and 2.4G WiFi seem to be working well. Switching back and forth between them seems timely.
