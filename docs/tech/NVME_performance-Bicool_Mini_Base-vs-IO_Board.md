# Pi CM4 NVME performance: Bicool Mini Base vs. IO Board

## Purpose

Compare performance of an NVME SSD on the (Waveshare) Bicool Mini Base A vs. the official Compute module 4 IO Board. Testinig is on CM4 H/W that uses direct PCIe/NVME connections (no USB adapter.)

## Motivation:

I configured a recently received CM4 on a Mini Base and performed some intitial testing to establish correct operation. Initial tests produced lower than expected disk bakdwidth. But there are confounding factors that could affect the tests. The initial test was performed with the SSD installed on an IO Board using a PCIe/NVME adapter and a CM4 with 8GB RAM. The card was then moved to the Mini Base which had a CM4 with 4GB RAM. Possible confounding factors.

* 4GB vs 8GB CM4
* Cooling on the SSD due to mounting location.
* Debian vs. Ubuntu (Further comparisons were performed between Debian on the CM4/8GB and the CM4/4GB on the respective base boards.)
* Differing ZFS versions on the Debian vs. Ubuntu installations.

## Process

1. Install vanilla Debian Bookworm to the 256GB NVME SSD. The same SSD will be used for all testinig. It will be erased using `blkdiscard` prior to each installatioin and it will be trimmed prior to each disk benchmark.
1. Perform two benchmarks on the CM4/4GB on the Mini Base. Direct a small fan at the SSD to control temperature and note results including SSD temperature.
1. Evaluate.
1. Swap the NVME SSD to the IO Board with the CM4/8GB.

```text
time -p iozone -f /home/hbarta/iozone.tst -e -o -a -s 100M -r 4k -i 0 -i 1 -i 2
time -p fio --randrepeat=1 --ioengine=libaio --gtod_reduce=1 --directory=/home/hbarta/ --name="io 256GB SSSTC NVME CM4/4GB Mini Base" --filename=test.dat --bs=4k --iodepth=64 --size=4G --readwrite=randrw --rwmixread=75
time -p fio --randrepeat=1 --ioengine=libaio --gtod_reduce=1 --directory=/home/hbarta/ --name="io 256GB SSSTC NVME CM4/8GB IO Board" --filename=test.dat --bs=4k --iodepth=64 --size=4G --readwrite=randrw --rwmixread=75
```

### Testinig on 

```text
hbarta@io:~$ time -p iozone -f /home/hbarta/iozone.tst -e -o -a -s 100M -r 4k -i 0 -i 1 -i 2
        Iozone: Performance Test of File I/O
                Version $Revision: 3.489 $
                Compiled for 64 bit mode.
                Build: linux 

        Contributors:William Norcott, Don Capps, Isom Crawford, Kirby Collins
                     Al Slater, Scott Rhine, Mike Wisner, Ken Goss
                     Steve Landherr, Brad Smith, Mark Kelly, Dr. Alain CYR,
                     Randy Dunlap, Mark Montague, Dan Million, Gavin Brebner,
                     Jean-Marc Zucconi, Jeff Blomberg, Benny Halevy, Dave Boone,
                     Erik Habbinga, Kris Strecker, Walter Wong, Joshua Root,
                     Fabrice Bacchella, Zhenghua Xue, Qin Li, Darren Sawyer,
                     Vangel Bojaxhi, Ben England, Vikentsi Lapa,
                     Alexey Skidanov, Sudhir Kumar.

        Run began: Tue Oct  3 18:02:42 2023

        Include fsync in write timing
        SYNC Mode. 
        Auto Mode
        File size set to 102400 kB
        Record Size 4 kB
        Command line used: iozone -f /home/hbarta/iozone.tst -e -o -a -s 100M -r 4k -i 0 -i 1 -i 2
        Output is in kBytes/sec
        Time Resolution = 0.000001 seconds.
        Processor cache size set to 1024 kBytes.
        Processor cache line size set to 32 bytes.
        File stride size set to 17 * record size.
                                                              random    random     bkwd    record    stride                                    
              kB  reclen    write  rewrite    read    reread    read     write     read   rewrite      read   fwrite frewrite    fread  freread
          102400       4     2781     4202   996001  1008897   986084     4180                                                                

iozone test complete.
real 86.36
user 0.41
sys 7.08
hbarta@io:~$ sudo fstrim -v /
[sudo] password for hbarta: 
/: 291.3 MiB (305418240 bytes) trimmed
hbarta@io:~$ time -p iozone -f /home/hbarta/iozone.tst -e -o -a -s 100M -r 4k -i 0 -i 1 -i 2
        Iozone: Performance Test of File I/O
                Version $Revision: 3.489 $
                Compiled for 64 bit mode.
                Build: linux 

        Contributors:William Norcott, Don Capps, Isom Crawford, Kirby Collins
                     Al Slater, Scott Rhine, Mike Wisner, Ken Goss
                     Steve Landherr, Brad Smith, Mark Kelly, Dr. Alain CYR,
                     Randy Dunlap, Mark Montague, Dan Million, Gavin Brebner,
                     Jean-Marc Zucconi, Jeff Blomberg, Benny Halevy, Dave Boone,
                     Erik Habbinga, Kris Strecker, Walter Wong, Joshua Root,
                     Fabrice Bacchella, Zhenghua Xue, Qin Li, Darren Sawyer,
                     Vangel Bojaxhi, Ben England, Vikentsi Lapa,
                     Alexey Skidanov, Sudhir Kumar.

        Run began: Tue Oct  3 18:04:48 2023

        Include fsync in write timing
        SYNC Mode. 
        Auto Mode
        File size set to 102400 kB
        Record Size 4 kB
        Command line used: iozone -f /home/hbarta/iozone.tst -e -o -a -s 100M -r 4k -i 0 -i 1 -i 2
        Output is in kBytes/sec
        Time Resolution = 0.000001 seconds.
        Processor cache size set to 1024 kBytes.
        Processor cache line size set to 32 bytes.
        File stride size set to 17 * record size.
                                                              random    random     bkwd    record    stride                                    
              kB  reclen    write  rewrite    read    reread    read     write     read   rewrite      read   fwrite frewrite    fread  freread
          102400       4     2824     4204   972903  1030087   992526     4181                                                                

iozone test complete.
real 85.77
user 0.45
sys 6.91
hbarta@io:~$ 
```

SSD temperature started at 30°C and ranged up to 35°C.

```text
hbarta@io:~$ sudo fstrim -v /
/: 217.6 MiB (228143104 bytes) trimmed
hbarta@io:~$ time -p fio --randrepeat=1 --ioengine=libaio --gtod_reduce=1 --directory=/home/hbarta/ --name="io 256GB SSSTC NVME CM4/4GB Mini Base" --filename=test.dat --bs=4k --iodepth=64 --size=4G --readwrite=randrw --rwmixread=75
io 256GB SSSTC NVME CM4/4GB Mini Base: (g=0): rw=randrw, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=64
fio-3.33
Starting 1 process
io 256GB SSSTC NVME CM4/4GB Mini Base: Laying out IO file (1 file / 4096MiB)
Jobs: 1 (f=1): [m(1)][100.0%][r=28.2MiB/s,w=9512KiB/s][r=7220,w=2378 IOPS][eta 00m:00s]
io 256GB SSSTC NVME CM4/4GB Mini Base: (groupid=0, jobs=1): err= 0: pid=3879: Tue Oct  3 18:09:33 2023
  read: IOPS=6987, BW=27.3MiB/s (28.6MB/s)(3070MiB/112478msec)
   bw (  KiB/s): min= 7048, max=35136, per=100.00%, avg=27977.00, stdev=3829.51, samples=224
   iops        : min= 1762, max= 8784, avg=6994.25, stdev=957.38, samples=224
  write: IOPS=2335, BW=9341KiB/s (9565kB/s)(1026MiB/112478msec); 0 zone resets
   bw (  KiB/s): min= 2368, max=11656, per=100.00%, avg=9350.50, stdev=1294.02, samples=224
   iops        : min=  592, max= 2914, avg=2337.63, stdev=323.50, samples=224
  cpu          : usr=6.07%, sys=31.19%, ctx=786476, majf=0, minf=19
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued rwts: total=785920,262656,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
   READ: bw=27.3MiB/s (28.6MB/s), 27.3MiB/s-27.3MiB/s (28.6MB/s-28.6MB/s), io=3070MiB (3219MB), run=112478-112478msec
  WRITE: bw=9341KiB/s (9565kB/s), 9341KiB/s-9341KiB/s (9565kB/s-9565kB/s), io=1026MiB (1076MB), run=112478-112478msec

Disk stats (read/write):
  nvme0n1: ios=785907/179450, merge=0/27, ticks=76811/91049, in_queue=167872, util=100.00%
real 127.56
user 9.10
sys 47.59
hbarta@io:~$ sudo fstrim -v /
/: 0 B (0 bytes) trimmed
hbarta@io:~$ time -p fio --randrepeat=1 --ioengine=libaio --gtod_reduce=1 --directory=/home/hbarta/ --name="io 256GB SSSTC NVME CM4/4GB Mini Base" --filename=test.dat --bs=4k --iodepth=64 --size=4G --readwrite=randrw --rwmixread=75
io 256GB SSSTC NVME CM4/4GB Mini Base: (g=0): rw=randrw, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=64
fio-3.33
Starting 1 process
Jobs: 1 (f=1): [m(1)][100.0%][r=25.2MiB/s,w=8752KiB/s][r=6442,w=2188 IOPS][eta 00m:00s]
io 256GB SSSTC NVME CM4/4GB Mini Base: (groupid=0, jobs=1): err= 0: pid=4012: Tue Oct  3 18:12:22 2023
  read: IOPS=6171, BW=24.1MiB/s (25.3MB/s)(3070MiB/127349msec)
   bw (  KiB/s): min= 6744, max=28472, per=100.00%, avg=25109.00, stdev=3328.02, samples=250
   iops        : min= 1686, max= 7118, avg=6277.23, stdev=832.00, samples=250
  write: IOPS=2062, BW=8250KiB/s (8448kB/s)(1026MiB/127349msec); 0 zone resets
   bw (  KiB/s): min= 2232, max=10184, per=100.00%, avg=8391.55, stdev=1142.27, samples=250
   iops        : min=  558, max= 2546, avg=2097.87, stdev=285.58, samples=250
  cpu          : usr=5.29%, sys=29.41%, ctx=786636, majf=0, minf=19
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued rwts: total=785920,262656,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
   READ: bw=24.1MiB/s (25.3MB/s), 24.1MiB/s-24.1MiB/s (25.3MB/s-25.3MB/s), io=3070MiB (3219MB), run=127349-127349msec
  WRITE: bw=8250KiB/s (8448kB/s), 8250KiB/s-8250KiB/s (8448kB/s-8448kB/s), io=1026MiB (1076MB), run=127349-127349msec

Disk stats (read/write):
  nvme0n1: ios=784644/184516, merge=0/32, ticks=89281/97942, in_queue=187238, util=98.44%
real 127.78
user 7.14
sys 37.96
hbarta@io:~$ 
```
SSD temperature hit a high of 39°C.

Results were considerably better than when running Ubuntu on ZFS (without cooling). Next step will be to move the SSD to the IO Board (which presently has a CM4/8GB installed) and repeat the benchmark.

### Repeat testing using CM4/8GB on IO Board

```text
hbarta@io:~$ time -p iozone -f /home/hbarta/iozone.tst -e -o -a -s 100M -r 4k -i 0 -i 1 -i 2
        Iozone: Performance Test of File I/O
                Version $Revision: 3.489 $
                Compiled for 64 bit mode.
                Build: linux 

        Contributors:William Norcott, Don Capps, Isom Crawford, Kirby Collins
                     Al Slater, Scott Rhine, Mike Wisner, Ken Goss
                     Steve Landherr, Brad Smith, Mark Kelly, Dr. Alain CYR,
                     Randy Dunlap, Mark Montague, Dan Million, Gavin Brebner,
                     Jean-Marc Zucconi, Jeff Blomberg, Benny Halevy, Dave Boone,
                     Erik Habbinga, Kris Strecker, Walter Wong, Joshua Root,
                     Fabrice Bacchella, Zhenghua Xue, Qin Li, Darren Sawyer,
                     Vangel Bojaxhi, Ben England, Vikentsi Lapa,
                     Alexey Skidanov, Sudhir Kumar.

        Run began: Tue Oct  3 22:11:09 2023

        Include fsync in write timing
        SYNC Mode. 
        Auto Mode
        File size set to 102400 kB
        Record Size 4 kB
        Command line used: iozone -f /home/hbarta/iozone.tst -e -o -a -s 100M -r 4k -i 0 -i 1 -i 2
        Output is in kBytes/sec
        Time Resolution = 0.000001 seconds.
        Processor cache size set to 1024 kBytes.
        Processor cache line size set to 32 bytes.
        File stride size set to 17 * record size.
                                                              random    random     bkwd    record    stride                                    
              kB  reclen    write  rewrite    read    reread    read     write     read   rewrite      read   fwrite frewrite    fread  freread
          102400       4     2707     4187  1107793  1098466   992342     4187                                                                

iozone test complete.
real 87.40
user 0.47
sys 7.17
hbarta@io:~$ sudo fstrim /
[sudo] password for hbarta: 
Sorry, try again.
[sudo] password for hbarta: 
hbarta@io:~$ time -p iozone -f /home/hbarta/iozone.tst -e -o -a -s 100M -r 4k -i 0 -i 1 -i 2
        Iozone: Performance Test of File I/O
                Version $Revision: 3.489 $
                Compiled for 64 bit mode.
                Build: linux 

        Contributors:William Norcott, Don Capps, Isom Crawford, Kirby Collins
                     Al Slater, Scott Rhine, Mike Wisner, Ken Goss
                     Steve Landherr, Brad Smith, Mark Kelly, Dr. Alain CYR,
                     Randy Dunlap, Mark Montague, Dan Million, Gavin Brebner,
                     Jean-Marc Zucconi, Jeff Blomberg, Benny Halevy, Dave Boone,
                     Erik Habbinga, Kris Strecker, Walter Wong, Joshua Root,
                     Fabrice Bacchella, Zhenghua Xue, Qin Li, Darren Sawyer,
                     Vangel Bojaxhi, Ben England, Vikentsi Lapa,
                     Alexey Skidanov, Sudhir Kumar.

        Run began: Tue Oct  3 22:14:17 2023

        Include fsync in write timing
        SYNC Mode. 
        Auto Mode
        File size set to 102400 kB
        Record Size 4 kB
        Command line used: iozone -f /home/hbarta/iozone.tst -e -o -a -s 100M -r 4k -i 0 -i 1 -i 2
        Output is in kBytes/sec
        Time Resolution = 0.000001 seconds.
        Processor cache size set to 1024 kBytes.
        Processor cache line size set to 32 bytes.
        File stride size set to 17 * record size.
                                                              random    random     bkwd    record    stride                                    
              kB  reclen    write  rewrite    read    reread    read     write     read   rewrite      read   fwrite frewrite    fread  freread
          102400       4     2771     4184  1084930  1077146   988608     4175                                                                

iozone test complete.
real 86.62
user 0.43
sys 7.26
hbarta@io:~$ 
```

SSD Temperature up to about 34°C.

```text
hbarta@io:~$ time -p fio --randrepeat=1 --ioengine=libaio --gtod_reduce=1 --directory=/home/hbarta/ --name="io 256GB SSSTC NVME CM4/8GB IO Board" --filename=test.dat --bs=4k --iodepth=64 --size=4G --readwrite=randrw --rwmixread=75
io 256GB SSSTC NVME CM4/8GB IO Board: (g=0): rw=randrw, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=64
fio-3.33
Starting 1 process
Jobs: 1 (f=1): [m(1)][100.0%][r=25.5MiB/s,w=8900KiB/s][r=6539,w=2225 IOPS][eta 00m:00s]
io 256GB SSSTC NVME CM4/8GB IO Board: (groupid=0, jobs=1): err= 0: pid=1042: Tue Oct  3 22:20:31 2023
  read: IOPS=5991, BW=23.4MiB/s (24.5MB/s)(3070MiB/131174msec)
   bw (  KiB/s): min= 6872, max=26364, per=100.00%, avg=23992.80, stdev=3362.60, samples=262
   iops        : min= 1718, max= 6591, avg=5998.15, stdev=840.66, samples=262
  write: IOPS=2002, BW=8009KiB/s (8202kB/s)(1026MiB/131174msec); 0 zone resets
   bw (  KiB/s): min= 2272, max= 9328, per=100.00%, avg=8019.03, stdev=1150.93, samples=262
   iops        : min=  568, max= 2332, avg=2004.74, stdev=287.74, samples=262
  cpu          : usr=5.06%, sys=26.46%, ctx=786433, majf=0, minf=18
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued rwts: total=785920,262656,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
   READ: bw=23.4MiB/s (24.5MB/s), 23.4MiB/s-23.4MiB/s (24.5MB/s-24.5MB/s), io=3070MiB (3219MB), run=131174-131174msec
  WRITE: bw=8009KiB/s (8202kB/s), 8009KiB/s-8009KiB/s (8202kB/s-8202kB/s), io=1026MiB (1076MB), run=131174-131174msec

Disk stats (read/write):
  nvme0n1: ios=785656/234183, merge=0/36, ticks=95721/121855, in_queue=217591, util=100.00%
real 131.68
user 6.99
sys 35.37
hbarta@io:~$ sudo fstrim /
hbarta@io:~$ time -p fio --randrepeat=1 --ioengine=libaio --gtod_reduce=1 --directory=/home/hbarta/ --name="io 256GB SSSTC NVME CM4/8GB IO Board" --filename=test.dat --bs=4k --iodepth=64 --size=4G --readwrite=randrw --rwmixread=75
io 256GB SSSTC NVME CM4/8GB IO Board: (g=0): rw=randrw, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=64
fio-3.33
Starting 1 process
Jobs: 1 (f=1): [m(1)][100.0%][r=25.3MiB/s,w=8596KiB/s][r=6483,w=2149 IOPS][eta 00m:00s]
io 256GB SSSTC NVME CM4/8GB IO Board: (groupid=0, jobs=1): err= 0: pid=1215: Tue Oct  3 22:23:52 2023
  read: IOPS=6044, BW=23.6MiB/s (24.8MB/s)(3070MiB/130019msec)
   bw (  KiB/s): min= 7344, max=26000, per=100.00%, avg=24581.84, stdev=3220.57, samples=255
   iops        : min= 1836, max= 6500, avg=6145.43, stdev=805.13, samples=255
  write: IOPS=2020, BW=8081KiB/s (8274kB/s)(1026MiB/130019msec); 0 zone resets
   bw (  KiB/s): min= 2264, max= 9296, per=100.00%, avg=8216.15, stdev=1117.15, samples=255
   iops        : min=  566, max= 2324, avg=2054.02, stdev=279.29, samples=255
  cpu          : usr=5.28%, sys=28.31%, ctx=786591, majf=0, minf=18
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued rwts: total=785920,262656,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
   READ: bw=23.6MiB/s (24.8MB/s), 23.6MiB/s-23.6MiB/s (24.8MB/s-24.8MB/s), io=3070MiB (3219MB), run=130019-130019msec
  WRITE: bw=8081KiB/s (8274kB/s), 8081KiB/s-8081KiB/s (8274kB/s-8274kB/s), io=1026MiB (1076MB), run=130019-130019msec

Disk stats (read/write):
  nvme0n1: ios=785133/182571, merge=0/30, ticks=92316/93259, in_queue=185591, util=98.26%
real 130.42
user 7.28
sys 37.34
hbarta@io:~$ 
```

Using elapsed time as a crude approximation of the benchmark result:

|CM4|base|benchmark|ET|
|---|---|---|---|
|4GB|Mini Base|iozone|86.36|
|4GB|Mini Base|iozone|85.77|
|4GB|Mini Base|fio|127.56|
|4GB|Mini Base|fio|127.78|
|8GB|IO Board|iozone|87.40|
|8GB|IO Board|iozone|86.62|
|8GB|IO Board|fio|131.68|
|8GB|IO Board|fio|130.42|

## Interpretation

The results are close enough to not warrant further investigation.
