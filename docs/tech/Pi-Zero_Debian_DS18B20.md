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

<https://github.com/raspberrypi/linux/blob/rpi-6.6.y/arch/arm/boot/dts/overlays/w1-gpio-overlay.dts>

Compile:

```text
hbarta@ceres:~/Documents/w1$ dtc w1-gpio-overlay.dts -o w1-gpio-overlay.dtb
w1-gpio-overlay.dts:12.18-18.6: Warning (unit_address_vs_reg): /fragment@0/__overlay__/onewire@0: node has a unit name, but no reg or ranges property
w1-gpio-overlay.dts:25.23-29.6: Warning (unit_address_vs_reg): /fragment@1/__overlay__/w1_pins@0: node has a unit name, but no reg or ranges property
hbarta@ceres:~/Documents/w1$ ls -l
total 8
-rw-r--r-- 1 hbarta hbarta 909 Oct 23 15:56 w1-gpio-overlay.dtb
-rw-r--r-- 1 hbarta hbarta 699 Oct 23 15:55 w1-gpio-overlay.dts
hbarta@ceres:~/Documents/w1$ 

```

```text
[15:45] <ukleinek> HankB: you can test if it applies using `fdtoverlay -i /boot/firmware/yourmachine.dtb -o /tmp/lala.dtb /boot/firmware/overlays/w1-gpio.dtbo`
```

```text
fdtoverlay -i /boot/firmware/bcm2835-rpi-zero-w.dtb -o /tmp/lala.dtb /boot/firmware/overlays/w1-gpio-overlay.dtbo
```

`fdtoverlay` produces no errors or warnings.

```text
hbarta@ceres:~/Documents/w1$ sudo mkdir /boot/firmware/overlays/
hbarta@ceres:~/Documents/w1$ sudo cp w1-gpio-overlay.dtbo /boot/firmware/overlays/
hbarta@ceres:~/Documents/w1$ 
```

Does not seem to change anything. Try moving the `w1-gpio-overlay.dtbo` to `/boot/firmware`. Seems to make no difference. Moving it back and looking at modules. RpiOS shows

```text
hbarta@niwot:~ $ lsmod|grep -E "wire|w1"
w1_therm               17732  0
w1_gpio                 3165  0
wire                   32902  2 w1_gpio,w1_therm
cn                      6073  1 wire
hbarta@niwot:~ $ 
```

No joy

```text
root@ceres:/home/hbarta# modprobe w1_therm wire w1_gpio cn
root@ceres:/home/hbarta# cat /sys/kernel/debug/devices_deferred
root@ceres:/home/hbarta# ls /sys/bus/w1/devices/
root@ceres:/home/hbarta# 
```
