# Pi Zero/W with working DS18B20

Uses a 1-wire temperture sensor. Desire to get working on a Pi Zero W running Debian (not RpiOS.)

Following up on a discussion initiated on `#debian-raspberrypi`.

Fully up to date Debian install with the following kernel

```text
hbarta@ceres:~$ uname -a
Linux ceres 6.1.0-26-rpi #1 Debian 6.1.112-1 (2024-09-30) armv6l GNU/Linux
hbarta@ceres:~$
```

`ukleinek` suggests "bookworm kernel is fine. 6.1.94-1 is the first good kernel." so I shouyld be good here.

```text
root@ceres:~# cat /sys/kernel/debug/devices_deferred
root@ceres:~#

root@ceres:~# modprobe w1_therm

root@ceres:~# lsmod|grep -E "wire|w1"
w1_therm               28672  0
wire                   45056  1 w1_therm
root@ceres:~# 

root@ceres:~# ls -lR /sys/bus/w1/
/sys/bus/w1/:
total 0
drwxr-xr-x 2 root root    0 Oct 23 15:25 devices
drwxr-xr-x 4 root root    0 Oct 23 15:25 drivers
-rw-r--r-- 1 root root 4096 Oct 23 15:25 drivers_autoprobe
--w------- 1 root root 4096 Oct 23 15:25 drivers_probe
--w------- 1 root root 4096 Oct 23 15:25 uevent

/sys/bus/w1/devices:
total 0

/sys/bus/w1/drivers:
total 0
drwxr-xr-x 2 root root 0 Oct 23 15:25 w1_master_driver
drwxr-xr-x 2 root root 0 Oct 23 15:25 w1_slave_driver

/sys/bus/w1/drivers/w1_master_driver:
total 0
--w------- 1 root root 4096 Oct 23 15:26 bind
--w------- 1 root root 4096 Oct 23 15:25 uevent
--w------- 1 root root 4096 Oct 23 15:26 unbind

/sys/bus/w1/drivers/w1_slave_driver:
total 0
--w------- 1 root root 4096 Oct 23 15:26 bind
--w------- 1 root root 4096 Oct 23 15:25 uevent
--w------- 1 root root 4096 Oct 23 15:26 unbind
root@ceres:~# 
```

```text
ukleinek starrs at w1-gpio.dtbo # presume 'starts'
```

This one? <https://github.com/raspberrypi/firmware/blob/master/boot/overlays/w1-gpio.dtbo>
