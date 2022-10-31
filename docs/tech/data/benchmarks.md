# CM4 benchmarks

Results with the Raspberry Pi IO Board and CM4 Ether Board.

## 2022-10-31 Pi IO Board

### ping

```text
hbarta@cm4nvme:~ $ ping -c 10 olive
...
--- olive.localdomain ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9042ms
rtt min/avg/max/mdev = 0.146/0.191/0.227/0.024 ms
hbarta@cm4nvme:~ $ 
hbarta@cm4nvme:~ $ 
hbarta@cm4nvme:~ $ ping -c 10 oak
...
--- oak.localdomain ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9030ms
rtt min/avg/max/mdev = 0.214/0.270/0.551/0.094 ms
hbarta@cm4nvme:~ $ 
```

### iperf

`iperf3 -s` running on the server `oak`

```text
hbarta@cm4nvme:~ $ iperf3 -c oak --get-server-output --bidir
Connecting to host oak, port 5201
...
[ ID][Role] Interval           Transfer     Bitrate         Retr  Cwnd
[  5][TX-C]   0.00-1.00   sec   112 MBytes   937 Mbits/sec    0    594 KBytes       
[  7][RX-C]   0.00-1.00   sec   109 MBytes   912 Mbits/sec                  
[  5][TX-C]   1.00-2.00   sec   111 MBytes   933 Mbits/sec    0    658 KBytes       
[  7][RX-C]   1.00-2.00   sec   112 MBytes   937 Mbits/sec                  
[  5][TX-C]   2.00-3.00   sec   110 MBytes   923 Mbits/sec    0    690 KBytes       
[  7][RX-C]   2.00-3.00   sec   112 MBytes   937 Mbits/sec                  
[  5][TX-C]   3.00-4.00   sec   110 MBytes   923 Mbits/sec    0    690 KBytes       
[  7][RX-C]   3.00-4.00   sec   112 MBytes   937 Mbits/sec                  
[  5][TX-C]   4.00-5.00   sec   110 MBytes   923 Mbits/sec    0    690 KBytes       
[  7][RX-C]   4.00-5.00   sec   112 MBytes   937 Mbits/sec                  
[  5][TX-C]   5.00-6.00   sec   110 MBytes   923 Mbits/sec    0    762 KBytes       
[  7][RX-C]   5.00-6.00   sec   112 MBytes   937 Mbits/sec                  
[  5][TX-C]   6.00-7.00   sec   110 MBytes   923 Mbits/sec    0    762 KBytes       
[  7][RX-C]   6.00-7.00   sec   112 MBytes   938 Mbits/sec                  
[  5][TX-C]   7.00-8.00   sec   109 MBytes   912 Mbits/sec    0    800 KBytes       
[  7][RX-C]   7.00-8.00   sec   111 MBytes   933 Mbits/sec                  
[  5][TX-C]   8.00-9.00   sec   110 MBytes   923 Mbits/sec    0    880 KBytes       
[  7][RX-C]   8.00-9.00   sec   112 MBytes   936 Mbits/sec                  
[  5][TX-C]   9.00-10.00  sec   110 MBytes   923 Mbits/sec    0    880 KBytes       
[  7][RX-C]   9.00-10.00  sec   112 MBytes   937 Mbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID][Role] Interval           Transfer     Bitrate         Retr
[  5][TX-C]   0.00-10.00  sec  1.08 GBytes   924 Mbits/sec    0             sender
[  5][TX-C]   0.00-10.03  sec  1.07 GBytes   919 Mbits/sec                  receiver

Server output:
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
...
[ ID][Role] Interval           Transfer     Bitrate         Retr  Cwnd
[  5][RX-S]   0.00-1.00   sec   106 MBytes   890 Mbits/sec                  
[  8][TX-S]   0.00-1.00   sec   108 MBytes   903 Mbits/sec    0    576 KBytes       
[  5][RX-S]   1.00-2.00   sec   110 MBytes   923 Mbits/sec                  
[  8][TX-S]   1.00-2.00   sec   112 MBytes   943 Mbits/sec    0    576 KBytes       
[  5][RX-S]   2.00-3.00   sec   110 MBytes   923 Mbits/sec                  
[  8][TX-S]   2.00-3.00   sec   112 MBytes   937 Mbits/sec    0    604 KBytes       
[  5][RX-S]   3.00-4.00   sec   110 MBytes   923 Mbits/sec                  
[  8][TX-S]   3.00-4.00   sec   112 MBytes   938 Mbits/sec    0    604 KBytes       
[  5][RX-S]   4.00-5.00   sec   110 MBytes   923 Mbits/sec                  
[  8][TX-S]   4.00-5.00   sec   112 MBytes   938 Mbits/sec    0    604 KBytes       
[  5][RX-S]   5.00-6.00   sec   110 MBytes   924 Mbits/sec                  
[  8][TX-S]   5.00-6.00   sec   111 MBytes   931 Mbits/sec    0    666 KBytes       
[  5][RX-S]   6.00-7.00   sec   110 MBytes   923 Mbits/sec                  
[  8][TX-S]   6.00-7.00   sec   112 MBytes   944 Mbits/sec    0    666 KBytes       
[  5][RX-S]   7.00-8.00   sec   110 MBytes   919 Mbits/sec                  
[  8][TX-S]   7.00-8.00   sec   111 MBytes   933 Mbits/sec    0   1.14 MBytes       
[  5][RX-S]   8.00-9.00   sec   110 MBytes   920 Mbits/sec                  
[  8][TX-S]   8.00-9.00   sec   111 MBytes   933 Mbits/sec    0   1.26 MBytes       
[  5][RX-S]   9.00-10.00  sec   110 MBytes   920 Mbits/sec                  
[  8][TX-S]   9.00-10.00  sec   111 MBytes   933 Mbits/sec    0   1.26 MBytes       
[  5][RX-S]  10.00-10.03  sec  3.59 MBytes   919 Mbits/sec                  
[  8][TX-S]  10.00-10.03  sec  3.75 MBytes   961 Mbits/sec    0   1.26 MBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID][Role] Interval           Transfer     Bitrate         Retr
[  5][RX-S]   0.00-10.03  sec  1.07 GBytes   919 Mbits/sec                  receiver
[  8][TX-S]   0.00-10.03  sec  1.09 GBytes   933 Mbits/sec    0             sender

[  7][RX-C]   0.00-10.00  sec  1.09 GBytes   936 Mbits/sec    0             sender
[  7][RX-C]   0.00-10.03  sec  1.09 GBytes   931 Mbits/sec                  receiver

iperf Done.
hbarta@cm4nvme:~ $ 
```

### fio

```text
time -p fio --randrepeat=1 --ioengine=libaio --gtod_reduce=1 --name="CM4 Pi-EB EXT4" --filename=test.dat --bs=4k --iodepth=64 --size=4G --readwrite=randrw --rwmixread=75
```

EXT4

```text
hbarta@cm4nvme:~ $ time -p fio --randrepeat=1 --ioengine=libaio --gtod_reduce=1 --name="CM4 Pi-EB EXT4" --filename=test.dat --bs=4k --iodepth=64 --size=4G --readwrite=randrw --rwmixread=75
CM4 Pi-EB EXT4: (g=0): rw=randrw, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=64
fio-3.25
Starting 1 process
CM4 Pi-EB EXT4: Laying out IO file (1 file / 4096MiB)
Jobs: 1 (f=1): [m(1)][100.0%][r=38.3MiB/s,w=12.4MiB/s][r=9801,w=3178 IOPS][eta 00m:00s]
CM4 Pi-EB EXT4: (groupid=0, jobs=1): err= 0: pid=1445: Mon Oct 31 10:20:32 2022
  read: IOPS=9489, BW=37.1MiB/s (38.9MB/s)(3070MiB/82821msec)
   bw (  KiB/s): min= 4728, max=40432, per=100.00%, avg=37980.56, stdev=5866.60, samples=165
   iops        : min= 1182, max=10108, avg=9495.14, stdev=1466.65, samples=165
  write: IOPS=3171, BW=12.4MiB/s (12.0MB/s)(1026MiB/82821msec); 0 zone resets
   bw (  KiB/s): min= 1480, max=14096, per=100.00%, avg=12694.01, stdev=2011.83, samples=165
   iops        : min=  370, max= 3524, avg=3173.50, stdev=502.96, samples=165
  cpu          : usr=6.20%, sys=29.42%, ctx=787488, majf=0, minf=17
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued rwts: total=785920,262656,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
   READ: bw=37.1MiB/s (38.9MB/s), 37.1MiB/s-37.1MiB/s (38.9MB/s-38.9MB/s), io=3070MiB (3219MB), run=82821-82821msec
  WRITE: bw=12.4MiB/s (12.0MB/s), 12.4MiB/s-12.4MiB/s (12.0MB/s-12.0MB/s), io=1026MiB (1076MB), run=82821-82821msec

Disk stats (read/write):
  nvme0n1: ios=784533/186940, merge=0/54, ticks=57364/359411, in_queue=416789, util=99.98%
real 100.79
user 7.70
sys 38.06
hbarta@cm4nvme:~ $ time -p fio --randrepeat=1 --ioengine=libaio --gtod_reduce=1 --name="CM4 Pi-EB EXT4" --filename=test.dat --bs=4k --iodepth=64 --size=4G --readwrite=randrw --rwmixread=75
CM4 Pi-EB EXT4: (g=0): rw=randrw, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=64
fio-3.25
Starting 1 process
Jobs: 1 (f=1): [m(1)][100.0%][r=25.5MiB/s,w=8832KiB/s][r=6531,w=2208 IOPS][eta 00m:00s]
CM4 Pi-EB EXT4: (groupid=0, jobs=1): err= 0: pid=1454: Mon Oct 31 10:24:53 2022
  read: IOPS=6503, BW=25.4MiB/s (26.6MB/s)(3070MiB/120854msec)
   bw (  KiB/s): min= 4608, max=31464, per=100.00%, avg=26473.72, stdev=3710.27, samples=237
   iops        : min= 1152, max= 7866, avg=6618.44, stdev=927.58, samples=237
  write: IOPS=2173, BW=8693KiB/s (8902kB/s)(1026MiB/120854msec); 0 zone resets
   bw (  KiB/s): min= 1752, max=11112, per=100.00%, avg=8847.19, stdev=1268.95, samples=237
   iops        : min=  438, max= 2778, avg=2211.80, stdev=317.24, samples=237
  cpu          : usr=6.21%, sys=30.13%, ctx=786638, majf=0, minf=16
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued rwts: total=785920,262656,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
   READ: bw=25.4MiB/s (26.6MB/s), 25.4MiB/s-25.4MiB/s (26.6MB/s-26.6MB/s), io=3070MiB (3219MB), run=120854-120854msec
  WRITE: bw=8693KiB/s (8902kB/s), 8693KiB/s-8693KiB/s (8902kB/s-8902kB/s), io=1026MiB (1076MB), run=120854-120854msec

Disk stats (read/write):
  nvme0n1: ios=784370/194729, merge=0/24, ticks=83268/359720, in_queue=443006, util=98.19%
real 121.10
user 7.75
sys 36.82
hbarta@cm4nvme:~ $ 
```

Mildly interesting that the second run took 20% longer.

ZFS

```
hbarta@cm4nvme:/mnt/pool $ zpool list
The ZFS modules are not loaded.
Try running '/sbin/modprobe zfs' as root to load them.
hbarta@cm4nvme:/mnt/pool $ /sbin/modprobe zfs
modprobe: FATAL: Module zfs not found in directory /lib/modules/5.15.74-v8+
hbarta@cm4nvme:/mnt/pool $ 
```

Kernel update did not build the modules (though no DKMS errors were observed) so ZFS testing will not be performed at this time.

### dd

```text
hbarta@cm4nvme:~ $ time -p dd-benchmark.sh 
Creating test file
4096+0 records in
4096+0 records out
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 36.0792 s, 119 MB/s
reading back test file
4096+0 records in
4096+0 records out
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 11.0287 s, 389 MB/s
write performance
4096+0 records in
4096+0 records out
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 20.5971 s, 209 MB/s
4096+0 records in
4096+0 records out
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 29.0112 s, 148 MB/s
read performance
4096+0 records in
4096+0 records out
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 11.0306 s, 389 MB/s
4096+0 records in
4096+0 records out
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 11.0704 s, 388 MB/s
real 129.61
user 0.19
sys 96.71
hbarta@cm4nvme:~ $ time -p dd-benchmark.sh 
Creating test file
4096+0 records in
4096+0 records out
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 38.493 s, 112 MB/s
reading back test file
4096+0 records in
4096+0 records out
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 11.1339 s, 386 MB/s
write performance
4096+0 records in
4096+0 records out
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 20.2999 s, 212 MB/s
4096+0 records in
4096+0 records out
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 29.0837 s, 148 MB/s
read performance
4096+0 records in
4096+0 records out
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 11.0611 s, 388 MB/s
4096+0 records in
4096+0 records out
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 11.0786 s, 388 MB/s
real 129.96
user 0.18
sys 95.28
hbarta@cm4nvme:~ $ 
```
