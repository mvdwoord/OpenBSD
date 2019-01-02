# 2019 is the year of OpenBSD on the desktop!
for me.

This repo serves as an incoherent, inconsistent, and probably also partially incorrect notes wihle I am using OpenBSD on a ThinkPad X1 Carbon (3rd Gen.)

## Setup notes
Setup documentation is excellent, things I learned along the way include:

- disklabels co-exist with fdisk partitions. I was fooled for a moment thinking the disklabels exist *within* an fdisk partition. Some disklabels are reserved:
    - *c* is for the entire disk (I think this is the only one that is really hardcoded. The others are more by convention)
    - *b* is for swap
    - *i* and onwards is used to map any existing non-OpenBSD (A6) partitions on the disk
    - I put / on *a* and /home on *h*
- Creating the softraid encrypted volume while you're installing from a USB stick led to another brief confusion. Internal HDD is mapped as /dev/sd0, the USB gets /dev/sd1 so your encrypted volume gets /dev/sd2. I was worried this would mess up after rebooting without USB as this wold swap the order. One of many pleasant surprises was that this is automatically corrected. My guess is this is based on the disk identifier.

```
guapo# sysctl hw.disknames
hw.disknames=sd0:9bf5bc14e0e5e7c8,sd1:c7e7509c73287187
```

- When re-installing, if you leave the mountpoint empty for the disklabel holding `/home` it won't be overwritten.. perfect when going through the setup a number of times to figure stuff out. Or semi-clean re-install keeping the homedirectory intact.

I'd like to dig in the whole MBR/GPT/disklabel a bit more though. I'd like to understand better how what is written where to disk. There is tons of legacy and compatibility issues in BIOS/UEFI land. I was digging around in the bootloader already but should make some test configurations with a USB stick and compare hexdumps. [This looks like a good start](https://superuser.com/a/354557)

Grabbing the boot sector like so:
```
guapo# dd if=/dev/sd0c of=/tmp/bootsector.bin bs=512 count=1
1+0 records in
1+0 records out
512 bytes transferred in 0.000 secs (1924913 bytes/sec)
guapo# hexdump /tmp/bootsector.bin
0000000    05ea    c000    8c07    8ec8    bcd0    fffc    d88e    a0b8
0000010    8e07    31c0    31f6    b9ff    0200    f3fc    eaa4    0022
0000020    07a0    071e    1f0e    02b4    16cd    03a8    0d74    07b0
0000030    dee8    6700    0d80    01b4    0000    f601    80c2    0875
0000040    49be    e801    00bf    80b2    bebe    b901    0004    048a
0000050    803c    0f74    c683    e210    bef5    017d    a6e8    fb00
0000060    ebf4    88fc    24d0    040f    a230    013a    34b0    c828
0000070    47a2    5601    2dbe    6701    05f6    01b4    0000    7501
0000080    4601    80e8    5e00    6726    05c7    01fe    0000    0000
0000090    f667    b405    0001    0100    3475    1488    aabb    b455
00000a0    cd41    8a13    7214    8127    55fb    75aa    f621    01c1
00000b0    1c74    2eb0    5ae8    6600    4c8b    6708    8966    250d
00000c0    0001    5600    42b4    1dbe    cd01    5e13    1a73    3bb0
00000d0    3ee8    8a00    0174    4c8b    b802    0201    db31    13cd
00000e0    0673    65be    e901    ff74    90be    e801    0017    6726
00000f0    3d81    01fe    0000    aa55    0575    00ea    007c    be00
0000100    0174    57e9    50ff    acfc    c084    0f74    02e8    eb00
0000110    50f6    b453    bb0e    0001    10cd    585b    10c3    0100
0000120    0000    c000    0007    0000    0000    0000    2100    7355
0000130    6e69    2067    7264    7669    2065    2c58    7020    7261
0000140    6974    6974    6e6f    5920    4d00    5242    6f20    206e
0000150    6c66    706f    7970    6f20    2072    6c6f    2064    4942
0000160    534f    0a0d    0d00    520a    6165    2064    7265    6f72
0000170    0d72    000a    6f4e    4f20    532f    0a0d    4e00    206f
0000180    6361    6974    6576    7020    7261    6974    6974    6e6f
0000190    0a0d    9000    0000    0000    0000    0000    0000    0000
00001a0    0000    0000    0000    0000    0000    0000    0000    0000
00001b0    0000    0000    0000    784f    0000    0000    0000    0000
00001c0    0000    0000    0000    0000    0000    0000    0000    0000
*
00001e0    0000    0000    0000    0000    0000    0000    0000    0180
00001f0    0002    fea6    ffff    0040    0000    f8da    1dce    aa55
0000200
```

