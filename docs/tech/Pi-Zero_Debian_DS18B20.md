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

`file` describes the `.dtbo`

```text
root@ceres:~# file /boot/firmware/overlays/w1-gpio-overlay.dtbo 
/boot/firmware/overlays/w1-gpio-overlay.dtbo: Device Tree Blob version 17, size=909, boot CPU=0, string block size=129, DT structure block size=724
root@ceres:~# 
```

## 2024-10-24 further direction via IRC

```text
[23:28] <ukleinek> HankB: /boot/firmware/overlays is irrelevant. That only matters if the kernel applies an overlay. You however want the overlay applied by the bootloader.
[23:29] <ukleinek> HankB: so place the overlay in /boot/firmware/overlays and add dtoverlay=w1-gpio to config.txt
[23:31] <ukleinek> If that worked you should have a dir /sys/firmware/devicetree/base/onewire@0
...
[03:19] <sur5r> ukleinek: those sentences contradict themselves. "/boot/firmware/overlays is irrelevant" <-> "place the overlay in /boot/firmware/overlays and add dtoverlay=w1-gpio to config.txt"
[03:20] <ukleinek> oh, indeed, it's /sys/firmware/devicetree/overlays that is irrelevant
[03:21] <ukleinek> I missread that and my copy-and-paste didn't follow my misinterpretation.
[03:23] <sur5r> I don't have /sys/firmware/devicetree/overlays
[03:23] <sur5r> Is that an RpiOS thing?
[03:24] <sur5r> I would love to be able to load DTBOs at runtime but never understood how to do that on Debian.
[03:26] <ukleinek> sur5r: there is a patch set for dynamic dtbo application from userspace, but no chance to get that into mainline. Many drivers assume the dtb doesn't change and keep references to it. Even if they maintain pointers to unrelated parts of the dtb their offset might change (I think), so that's calling for data corruption.
[03:27] <ukleinek> https://git.kernel.org/pub/scm/linux/kernel/git/geert/renesas-drivers.git/log/?h=topic/overlays is the most maintained version of if AFAIK
[03:30] <ukleinek> There is an in-kernel API though, and if you use that, /sys/firmware/devicetree/overlays is created. (Dangerous half knowledge. That is at least the reason the dtb is in /sys/firmware/devicetree/base and not in /sys/firmware/devicetree/ directly)
[03:32] <jochensp> ukleinek: do you mean CONFIG_OF_DYNAMIC?
[03:32] <ukleinek> Yes, that's the in-kernel one
[03:35] <ukleinek> ah, /sys/firmware/devicetree/overlays only comes into play with the above mentioned patches applied.
[03:47] <sur5r> Yeah I remember the RaspiOS tool warns loudly when trying to remove dtbos
[05:24] <sur5r> *sigh* fbtft can't be built OOT against the Debian kernel as it needs FB_BACKLIGHT enabled
```

```text
root@ceres:~# tail /boot/firmware/config.txt 
enable_uart=1
upstream_kernel=1

kernel=vmlinuz-6.1.0-26-rpi
# For details on the initramfs directive, see
# https://www.raspberrypi.org/forums/viewtopic.php?f=63&t=10532
initramfs initrd.img-6.1.0-26-rpi

#dtoverlay=w1-gpio.gpiopin=4
dtoverlay=w1-gpio
root@ceres:~# ls -l /sys/firmware/devicetree/overlays
ls: cannot access '/sys/firmware/devicetree/overlays': No such file or directory
root@ceres:~# ls -l /sys/firmware/devicetree/        
total 0
drwxr-xr-x 17 root root 0 Aug 25 12:35 base
root@ceres:~# ls -l /sys/firmware/devicetree/base
total 0
-r--r--r--  1 root root  4 Oct 24 09:43 '#address-cells'
-r--r--r--  1 root root  4 Oct 24 09:43 '#size-cells'
drwxr-xr-x  2 root root  0 Oct 24 09:43  __symbols__
drwxr-xr-x  2 root root  0 Oct 24 09:43  aliases
drwxr-xr-x  2 root root  0 Oct 24 09:43  arm-pmu
drwxr-xr-x  3 root root  0 Oct 24 09:43  axi
drwxr-xr-x  4 root root  0 Oct 24 09:43  chosen
drwxr-xr-x  4 root root  0 Oct 24 09:43  clocks
-r--r--r--  1 root root 38 Aug 25 12:35  compatible
drwxr-xr-x  3 root root  0 Oct 24 09:43  cpus
-r--r--r--  1 root root  4 Oct 24 09:43  interrupt-parent
drwxr-xr-x  3 root root  0 Oct 24 09:43  leds
drwxr-xr-x  2 root root  0 Oct 24 09:43  memory@0
-r--r--r--  1 root root  8 Oct 24 09:43  memreserve
-r--r--r--  1 root root 28 Oct 24 09:43  model
-r--r--r--  1 root root  1 Oct 24 09:43  name
drwxr-xr-x  2 root root  0 Oct 24 09:43  phy
drwxr-xr-x  3 root root  0 Oct 24 09:43  reserved-memory
-r--r--r--  1 root root 17 Oct 24 09:43  serial-number
drwxr-xr-x 40 root root  0 Oct 24 09:43  soc
drwxr-xr-x  2 root root  0 Oct 24 09:43  system
drwxr-xr-x  3 root root  0 Oct 24 09:43  thermal-zones
drwxr-xr-x  2 root root  0 Oct 24 09:43  wifi-pwrseq
root@ceres:~# 
```

Reboot now

```text
root@ceres:~# ls -l /sys/firmware/devicetree/overlays
ls: cannot access '/sys/firmware/devicetree/overlays': No such file or directory
root@ceres:~# tail -1 /boot/firmware/config.txt 
dtoverlay=w1-gpio
root@ceres:~# for m in  w1_therm wire w1_gpio cn; do modprobe $m; done
root@ceres:~# ls -l /sys/firmware/devicetree/overlays
ls: cannot access '/sys/firmware/devicetree/overlays': No such file or directory
root@ceres:~# 
```

Further checking

```text
root@ceres:/home/hbarta# find /sys -name "onewire*"
root@ceres:/home/hbarta# find /sys -name "one*"
root@ceres:/home/hbarta# find /sys -name "wire*"
/sys/devices/platform/soc/20300000.mmc/mmc_host/mmc1/mmc1:0001/mmc1:0001:1/net/wlan0/wireless
/sys/module/wire
root@ceres:/home/hbarta# find /sys -name "w1*"  
/sys/bus/platform/drivers/w1-gpio
/sys/bus/w1
/sys/bus/w1/drivers/w1_slave_driver
/sys/bus/w1/drivers/w1_master_driver
/sys/module/w1_gpio
/sys/module/wire/holders/w1_gpio
/sys/module/wire/holders/w1_therm
/sys/module/w1_therm
root@ceres:/home/hbarta# 
```

## 2024-10-25 dtbo naming

```text
dtc w1-gpio-overlay.dts -o w1-gpio.dtbo
sudo cp w1-gpio.dtbo
# Check /boot/firmware/config.txt
```

```text
hbarta@ceres:~/Documents/w1$ dtc w1-gpio-overlay.dts -o w1-gpio.dtbo
w1-gpio-overlay.dts:12.18-18.6: Warning (unit_address_vs_reg): /fragment@0/__overlay__/onewire@0: node has a unit name, but no reg or ranges property
w1-gpio-overlay.dts:25.23-29.6: Warning (unit_address_vs_reg): /fragment@1/__overlay__/w1_pins@0: node has a unit name, but no reg or ranges property
hbarta@ceres:~/Documents/w1$ sudo cp w1-gpio.dtbo /boot/firmware/overlays
[sudo] password for hbarta: 
hbarta@ceres:~/Documents/w1$ tail -1 /boot/firmware/config.txt 
dtoverlay=w1-gpio
hbarta@ceres:~/Documents/w1$ sudo shutdown -r now
```

Following reboot:

```text
Linux ceres 6.1.0-26-rpi #1 Debian 6.1.112-1 (2024-09-30) armv6l
...
Last login: Fri Oct 25 08:30:42 2024 from 192.168.1.85
hbarta@ceres:~$ lsmod | grep w1     
w1_gpio                16384  0
wire                   45056  1 w1_gpio
hbarta@ceres:~$ ls /sys/bus/w1/devices/
hbarta@ceres:~$ cat /boot/firmware/config.txt
# Do not modify this file!
#
# It is automatically generated upon install or update of either the
# firmware or the Linux kernel.
#
# If you need to set boot-time parameters, do so via the
# /etc/default/raspi-firmware, /etc/default/raspi-firmware-custom or
# /etc/default/raspi-extra-cmdline files.

enable_uart=1
upstream_kernel=1

kernel=vmlinuz-6.1.0-26-rpi
# For details on the initramfs directive, see
# https://www.raspberrypi.org/forums/viewtopic.php?f=63&t=10532
initramfs initrd.img-6.1.0-26-rpi

#dtoverlay=w1-gpio.gpiopin=4
dtoverlay=w1-gpio
hbarta@ceres:~$ ls /boot/firmware/overlays
w1-gpio.dtbo
hbarta@ceres:~$ cat /sys/kernel/debug/devices_deferred
cat: /sys/kernel/debug/devices_deferred: Permission denied
hbarta@ceres:~$ sudo cat /sys/kernel/debug/devices_deferred
[sudo] password for hbarta: 
hbarta@ceres:~$ ls -l /sys/firmware/devicetree/overlays
ls: cannot access '/sys/firmware/devicetree/overlays': No such file or directory
hbarta@ceres:~$ 
hbarta@ceres:~$ sudo modprobe w1_therm
hbarta@ceres:~$ ls -l /sys/firmware/devicetree/overlays
ls: cannot access '/sys/firmware/devicetree/overlays': No such file or directory
hbarta@ceres:~$ ls /sys/bus/w1/devices/
hbarta@ceres:~$ lsmod | grep w1
w1_therm               28672  0
w1_gpio                16384  0
wire                   45056  2 w1_gpio,w1_therm
hbarta@ceres:~$ 
```

Difference: module `w1_gpio` loaded automatically. `w1_therm` did not.

```text
fdtoverlay -i /boot/firmware/bcm2835-rpi-zero-w.dtb -o /tmp/lala.dtb /boot/firmware/overlays/w1-gpio.dtbo
```

Produces no errors/warnings

### More

```text
[09:50] <jochensp> hbarta_: you could try the raspberry kernel dtb and then debug which part mainline is missing
[09:53] <hbarta_> I'll take a look at that - thx. I think I tried that at one point but might have fumbled something.
[09:54] <hbarta_> Actually, this is the DTS pulled from the RpiOS repo that I'm using.
[09:59] <jochensp> hbarta_: I mean the complete dtb, not only the overlay
[10:00] <-- chele (~chele@00022067.user.oftc.net) has left this server (Remote host closed the connection).
[10:00] <-- minimal (~minimal@0002b71e.user.oftc.net) has left this server (Quit: Leaving).
[10:06] <ukleinek> hbarta_: the warnings don't matter
[10:06] <ukleinek> s/warnings/& by dtc/
[10:07] <ukleinek> no, don't pollute you system with rpi stuff :-)
[10:08] <jochensp> ukleinek: feel free to propose a better way but that's how I found the missing peaces last time ;)
[10:08] <hbarta_> I don't kow what you mean by "complete dtb" Is that the contents of the directory I pulled the DTS from
[10:09] <hbarta_> The DTS I'm working with is from an Rpi repo.
[10:10] <ukleinek> hbarta_: dmesg | grep -E onewire
[10:11] * ukleinek expects a problem with pinctrl
[10:11] <ukleinek> hbarta_: jochensp suggests to replace the machine dtb.
[10:13] <ukleinek> "module w1_gpio loaded automatically. w1_therm did not." is not surprising.
[10:13] <hbarta_> I tried (what I believe) is the machine DTB from a Pi with 6.6 kernel and the Pi stuck at the rainbow screen.
[10:14] <ukleinek> because there is a w1-gpio device now (that however fails to probe because of a pinctrl conflict) and as the bus is missing there is no therm device that would trigger autoloading w1-therm
[10:15] <hbarta_> 3 lines from the grep dmesg
[10:16] <hbarta_> [   35.169615] pinctrl-bcm2835 20200000.gpio: pin gpio4 already requested by 20200000.gpio; cannot claim for onewire@0
[10:16] <hbarta_> [   35.195280] pinctrl-bcm2835 20200000.gpio: pin-4 (onewire@0) status -22
[10:16] <hbarta_> [   35.242269] w1-gpio onewire@0: Error applying setting, reverse things back
[10:16] <hbarta_> Seems significant.
[10:16] <ukleinek> it is
[10:17] <hbarta_> Can you suggest a fix?
[10:17] * ukleinek is working at it
[10:17] <hbarta_> Thx
```

Working! <https://paste.debian.net/hidden/d5ad73b2/>

```text
diff --git a/arch/arm/boot/dts/broadcom/bcm2835-rpi-zero-w.dts b/arch/arm/boot/dts/broadcom/bcm2835-rpi-zero-w.dts
index 1f0b163e400c..9ee4ffc1cd38 100644
--- a/arch/arm/boot/dts/broadcom/bcm2835-rpi-zero-w.dts
+++ b/arch/arm/boot/dts/broadcom/bcm2835-rpi-zero-w.dts
@@ -98,7 +98,7 @@ &gpio {
 			  "SD_DATA3_R";
 
 	pinctrl-names = "default";
-	pinctrl-0 = <&gpioout &alt0>;
+	pinctrl-0 = <&gpioout>;
 };
 
 &hdmi {
```

* Pi Zero <https://github.com/torvalds/linux/blob/master/arch/arm/boot/dts/broadcom/bcm2835-rpi-zero.dts>
* Pi Zero W<https://github.com/torvalds/linux/blob/master/arch/arm/boot/dts/broadcom/bcm2835-rpi-zero-w.dts>

(Note: `wget` the raw URL.)

I need to repeat this, perhaps for other Pi models to confirm I have the needed fixes and repeatable instructions.

Again, many thanks to the denizens of the #debian-raspberrypi who solved this for me.

## 2024-11-05 Works with RpiOS

```text
[16:40] <ukleinek> HankB: I would be interested if the w1-gpio overlay works for the pizero using rpios.
[16:43] <-- leighbb (~leighbb@2001:8b0:1df6:8fca:bab9:33:f4b0:28ef) has left this server (Quit: Leaving).
[16:43] <ukleinek> HankB: sounds as if you'd need access to your vanishing pizeros via an UART console. What do you use to configure the wifi connection? nm? networkd?
[16:44] <ukleinek> Maybe a reconnect issue? Did you try to disable the wifi shortly and then check if the device reconnect?
```

## 2024-11-21 repeat following kernel upgrade

* Not on a fresh install.
* On a Zero W host `ceres`.
* Version of DTB files seems to be the same (e.g. <bcm2835-rpi-...>)
* DS18B20 device no longer present.

```text
cd ~/GPIO
wget https://raw.githubusercontent.com/torvalds/linux/refs/heads/master/arch/arm/boot/dts/broadcom/bcm2835-rpi-zero-w.dts
dtc bcm2835-rpi-zero-w.dts -o bcm2835-rpi-zero-w.dtb
```

Last command produces the error 

```text
hbarta@ceres:~/GPIO$ dtc bcm2835-rpi-zero-w.dts
Error: bcm2835-rpi-zero-w.dts:7.1-9 syntax error
FATAL ERROR: Unable to parse input tree
hbarta@ceres:~/GPIO$ 
```

However, after adding `dtoverlay=w1-gpio` to `config.txt` the DS18B20 device is present (following reboot.)
