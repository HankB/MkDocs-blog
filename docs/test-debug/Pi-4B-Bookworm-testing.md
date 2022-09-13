# Pi 4B Bookworm testing

Determine if some previous bugs have been fixed in more recent kernels. Bugs fell into two areas:

* vc4 module - this included a use-after-free as well as other issues reported in `dmesg`.
* SD card timeout - Timeout message repeated every 10s. This had manifested previously and a workaround was to insert an SD card into the slot and the messages stopped. This time that does not work. Further characteristics of the present situation:
  * doesn't boot
  * Boots from attached USB SSD skipping the SD card.
  * No /dev/mmc* after boot accompanied by no WiFi.
  * /dev/mmc* entries, can mount and read card and after several more timeout messages, they stop.
  * When the timeout messages are being reported, the system cannot complete shutdown to reboot. It is necessary to remove power from the system.
  * Normal boot, no mmc timeouts and apparently normal operation.

In other words, there is a great variety of behavior with kernels with Bookworm. The good news is that later kernels do not exhibit the problems with the vc4 module so the focus on these notes is to better characterize the SD timeout issue now reported at <https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1019700>.


