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

WiFi works on 5G, does not associate on 2.4G. Captured output to `daemon.log` when trying to switch from an associated 5G AP to a 2.4G AP up until NM re-requested the WiFi password. <https://paste.debian.net/1236229/>

Connect times to the 5G AP was quick at times and not quick other times. At one point it was necessary to turn WiFi off and on again to get it to associate. (Both 5G and 2.4G radios are on the same WiFi AP.)

Did not shutdown cleanly. Had to pull power.

## 5.17.0-trunk-arm64

From experimental

```text
Linux up 5.17.0-trunk-arm64 #1 SMP Debian 5.17.1-1~exp1 (2022-03-29) aarch64 GNU/Linux
```

Tested a second time and system hung. Hadn't remoted in - working from desktop. Happened once before and was unable to restart GDM and `shutdown` hung too. Was able to examine logs and `dmesg` output and saw nothing unusual but may not have been looking in the right place. Had to remove power both times.

Both 5G and 2.4G WiFi seem to be working well. Switching back and forth between them seems timely.

## 2022-04-03 repeating testing.

### Setup: Recent install of bookworm install media.

```text
xzcat 20220121_raspi_4_bookworm.img.xz> /dev/sdf
```

Install `raspi-firmware` from unstable (direct download) and `network-manager` and full upgrade of all bookworm packages.

```text
apt upgrade -y
wget http://ftp.us.debian.org/debian/pool/non-free/r/raspi-firmware/raspi-firmware_1.20220328+ds-1_arm64.deb
apt install ./raspi-firmware_1.20220328+ds-1_arm64.deb
apt install network-manager
```

### Instructions for testing/managing WiFi using CLI

>HankB_, yes, from the command line, first, turn the wifi device on "nmcli radio wifi on" then list the wifi connections "nmcli dev wifi list", then sudo nmcli dev wifi connect network-ssid password "network-password"


Test with `5.17.0-trunk-arm64` (ssh in via Ethernet)

```text
root@charm:~# nmcli radio wifi on
root@charm:~# nmcli dev wifi list
IN-USE  BSSID              SSID                MODE   CHAN  RATE        SIGNAL  BARS  SECURITY 
        70:4F:57:11:58:63  Nacho24             Infra  6     130 Mbit/s  100     ****  WPA2     
        10:C3:7B:55:05:E8  Giant Voice System  Infra  10    195 Mbit/s  97      ****  WPA2     
        10:C3:7B:55:05:E9  NRO-33              Infra  10    195 Mbit/s  97      ****  WPA2     
        70:4F:57:11:58:62  Nacho5G             Infra  149   270 Mbit/s  77      ***   WPA2     
        10:C3:7B:55:05:EC  Fanghorn Forest     Infra  157   405 Mbit/s  77      ***   WPA2     
root@charm:~# nmcli dev wifi connect "Fanghorn Forest" password "elided"
Device 'wlan0' successfully activated with 'elided'.
root@charm:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether dc:a6:32:bf:65:b5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.103/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 5209sec preferred_lft 5209sec
    inet6 2601:249:1680:36f0:dea6:32ff:febf:65b5/64 scope global dynamic mngtmpaddr 
       valid_lft 86390sec preferred_lft 14390sec
    inet6 fe80::dea6:32ff:febf:65b5/64 scope link 
       valid_lft forever preferred_lft forever
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether dc:a6:32:bf:65:b6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.164/24 brd 192.168.1.255 scope global dynamic noprefixroute wlan0
       valid_lft 7194sec preferred_lft 7194sec
    inet6 2601:249:1680:36f0::1a72/128 scope global dynamic noprefixroute 
       valid_lft 7197sec preferred_lft 4497sec
    inet6 2601:249:1680:36f0:dd52:e503:3685:cb27/64 scope global dynamic noprefixroute 
       valid_lft 86397sec preferred_lft 14397sec
    inet6 fe80::add9:ac99:37f5:fddb/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
root@charm:~# 
root@charm:~# 
root@charm:~# 
root@charm:~# ifdown eth0
Killed old client process
Internet Systems Consortium DHCP Client 4.4.2-P1
Copyright 2004-2021 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/eth0/dc:a6:32:bf:65:b5
Sending on   LPF/eth0/dc:a6:32:bf:65:b5
Sending on   Socket/fallback
DHCPRELEASE of 192.168.1.103 on eth0 to 192.168.1.1 port 67
```

(This session was over Ethernet and resulted in a hung session following `ifdown eth0` but I felt that was the easiest way to confirm that traffic was actually going over WiFi.)

Reconnnect 

```text
Linux charm 5.17.0-trunk-arm64 #1 SMP Debian 5.17.1-1~exp1 (2022-03-29) aarch64 GNU/Linux
```

Kernels available

```text
root@charm:~# ls -l /boot/vmlinuz*
-rw-r--r-- 1 root root 28614528 Dec 18 23:20 /boot/vmlinuz-5.15.0-2-arm64
-rw-r--r-- 1 root root 29998976 Mar 15 06:54 /boot/vmlinuz-5.16.0-5-arm64
-rw-r--r-- 1 root root 30003072 Mar 29 07:16 /boot/vmlinuz-5.17.0-trunk-arm64
root@charm:~# 
```

Test with `5.15.0-2-arm64` (ssh in via Ethernet)

```text
root@charm:~# uname -a
Linux charm 5.15.0-2-arm64 #1 SMP Debian 5.15.5-2 (2021-12-18) aarch64 GNU/Linux
root@charm:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether dc:a6:32:bf:65:b5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.103/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 7136sec preferred_lft 7136sec
    inet6 2601:249:1680:36f0:dea6:32ff:febf:65b5/64 scope global dynamic mngtmpaddr 
       valid_lft 86394sec preferred_lft 14394sec
    inet6 fe80::dea6:32ff:febf:65b5/64 scope link 
       valid_lft forever preferred_lft forever
root@charm:~# nmcli radio wifi on
root@charm:~# nmcli dev wifi list
root@charm:~# nmcli dev wifi connect "Fanghorn Forest" password "elided"
Error: No Wi-Fi device found.
root@charm:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether dc:a6:32:bf:65:b5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.103/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 7098sec preferred_lft 7098sec
    inet6 2601:249:1680:36f0:dea6:32ff:febf:65b5/64 scope global dynamic mngtmpaddr 
       valid_lft 86397sec preferred_lft 14397sec
    inet6 fe80::dea6:32ff:febf:65b5/64 scope link 
       valid_lft forever preferred_lft forever
root@charm:~# 
```

dmesg paste at <https://paste.debian.net/1236660/>

### test with `5.16.0-5-arm64` starting with SSH over Ethernet

```text
root@charm:~# uname -a
Linux charm 5.16.0-5-arm64 #1 SMP Debian 5.16.14-1 (2022-03-15) aarch64 GNU/Linux
root@charm:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether dc:a6:32:bf:65:b5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.103/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 7155sec preferred_lft 7155sec
    inet6 2601:249:1680:36f0:dea6:32ff:febf:65b5/64 scope global dynamic mngtmpaddr 
       valid_lft 86389sec preferred_lft 14389sec
    inet6 fe80::dea6:32ff:febf:65b5/64 scope link 
       valid_lft forever preferred_lft forever
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether dc:a6:32:bf:65:b6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.164/24 brd 192.168.1.255 scope global dynamic noprefixroute wlan0
       valid_lft 5692sec preferred_lft 5692sec
    inet6 2601:249:1680:36f0::1a72/128 scope global dynamic noprefixroute 
       valid_lft 5696sec preferred_lft 2996sec
    inet6 2601:249:1680:36f0:dd52:e503:3685:cb27/64 scope global dynamic noprefixroute 
       valid_lft 86390sec preferred_lft 14390sec
    inet6 fe80::add9:ac99:37f5:fddb/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
root@charm:~# ifdown eth0
Killed old client process
Internet Systems Consortium DHCP Client 4.4.2-P1
Copyright 2004-2021 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/eth0/dc:a6:32:bf:65:b5
Sending on   LPF/eth0/dc:a6:32:bf:65:b5
Sending on   Socket/fallback
DHCPRELEASE of 192.168.1.103 on eth0 to 192.168.1.1 port 67
```

Start second session over WiFi

```text
hbarta@charm:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether dc:a6:32:bf:65:b5 brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether dc:a6:32:bf:65:b6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.164/24 brd 192.168.1.255 scope global dynamic noprefixroute wlan0
       valid_lft 5595sec preferred_lft 5595sec
    inet6 2601:249:1680:36f0::1a72/128 scope global dynamic noprefixroute 
       valid_lft 5599sec preferred_lft 2899sec
    inet6 2601:249:1680:36f0:dd52:e503:3685:cb27/64 scope global dynamic noprefixroute 
       valid_lft 86399sec preferred_lft 14399sec
    inet6 fe80::add9:ac99:37f5:fddb/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
hbarta@charm:~$ 
```

Confirmation that WiFi is working on both 5G ("Fanghorn Forest") and 2.4G (Giant Voice System) via WiFi ssh session.

```text
hbarta@charm:~$ nmcli dev wifi list 
IN-USE  BSSID              SSID                MODE   CHAN  RATE        SIGNAL  BARS  SECURITY 
        70:4F:57:11:58:63  Nacho24             Infra  6     130 Mbit/s  100     ****  WPA2     
        10:C3:7B:55:05:E9  NRO-33              Infra  10    195 Mbit/s  100     ****  WPA2     
        70:4F:57:11:58:62  Nacho5G             Infra  149   270 Mbit/s  84      ****  WPA2     
*       10:C3:7B:55:05:E8  Giant Voice System  Infra  10    195 Mbit/s  80      ***   WPA2     
        10:C3:7B:55:05:EC  Fanghorn Forest     Infra  157   405 Mbit/s  75      ***   WPA2     
hbarta@charm:~$ su -
root@charm:~# nmcli dev wifi connect "Fanghorn Forest"
Device 'wlan0' successfully activated with '369ee301-096a-420b-99a9-68def48160f9'.
root@charm:~# nmcli dev wifi list
IN-USE  BSSID              SSID                MODE   CHAN  RATE        SIGNAL  BARS  SECURITY 
        70:4F:57:11:58:63  Nacho24             Infra  6     130 Mbit/s  100     ****  WPA2     
        10:C3:7B:55:05:E9  NRO-33              Infra  10    195 Mbit/s  100     ****  WPA2     
        10:C3:7B:55:05:E8  Giant Voice System  Infra  10    195 Mbit/s  94      ****  WPA2     
        70:4F:57:11:58:62  Nacho5G             Infra  149   270 Mbit/s  79      ***   WPA2     
*       10:C3:7B:55:05:EC  Fanghorn Forest     Infra  157   405 Mbit/s  66      ***   WPA2     
root@charm:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether dc:a6:32:bf:65:b5 brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether dc:a6:32:bf:65:b6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.164/24 brd 192.168.1.255 scope global dynamic noprefixroute wlan0
       valid_lft 6825sec preferred_lft 6825sec
    inet6 2601:249:1680:36f0::1a72/128 scope global dynamic noprefixroute 
       valid_lft 6827sec preferred_lft 4127sec
    inet6 2601:249:1680:36f0:dd52:e503:3685:cb27/64 scope global dynamic noprefixroute 
       valid_lft 86401sec preferred_lft 14401sec
    inet6 fe80::add9:ac99:37f5:fddb/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
root@charm:~# uname -a
Linux charm 5.17.0-trunk-arm64 #1 SMP Debian 5.17.1-1~exp1 (2022-03-29) aarch64 GNU/Linux
root@charm:~# 
```
