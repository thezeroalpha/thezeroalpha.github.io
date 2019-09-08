---
layout: post
title: 'GPT: Recovering Partitions From a Disk With a Broken Partition Table'
author: zeroalpha
categories: Guide
date: 2019-09-08 12:27 -0400
---

I recently did a Linux installation onto my secondary hard drive, which also contains a FileVault-encrypted HFS+ volume.
Well, when I booted back into macOS, I found out that the volume was suddenly unreadable, and all of my books, movies, music, etc. were therefore gone.
What now?

A `diskutil list` showed something like:

```
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
...
   2:   FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF                    449.9 GB   disk0s2
...
```

Hoping for a quick recovery, I live-booted into SystemRescueCD (which, by the way, is a great tool for data recovery).
`parted` didn't see the partition, and `testdisk` reported that the partition was unrecoverable.
Great.

Since the disk uses a GUID Partition Table, I decided to try one more thing.
I booted back into macOS, and ran `gpt recover`, which usually finds the missing partitions.
However, this time I got an error message saying "suspicious mbr at sector 0".

Awesome.
Now I'm screwed.

I started doing some research, and (spoiler alert) I managed to get all of my data back.
But this is a process that I definitely won't remember, so I decided to put it into a blog post, in case I ever screw up this badly again (which I probably will).

For reference, the hard drive is `/dev/disk0` and the bad partition is `/dev/disk0s2`.
Also, everything has to either be prefixed with `sudo`, or you have to log in as super-user.
Finally, if at any point you get a message saying "unable to open device '/dev/disk0': Resource busy", run `diskutil unmountDisk disk0` before the command.

## How to fix the partition table
First, unmount the whole disk with `diskutil umountDisk disk0`.

Then, do `gpt -r show disk0` to get this kind of output:

```
gpt show: disk0: Suspicious MBR at sector 0
      start       size  index  contents
          0          1         MBR
          1          1         Pri GPT header
          2         32         Pri GPT table
         34          6
         40     409600      1  GPT part - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
     409640  878626472      2  GPT part - FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF
  879036112     262144      3  GPT part - 48465300-0000-11AA-AA11-00306543ECAC
  879298256     262448
  879560704   91762688      4  GPT part - 0FC63DAF-8483-4772-8E79-3D69D8477DE4
  971323392    1228799      5  GPT part - C12A7328-F81F-11D2-BA4B-00A0C93EC93B
  972552191          1
  972552192    4220927      6  GPT part - 0657FD6D-A4AB-43C4-84E5-0933C84B4F4F
  976773119         16
  976773135         32         Sec GPT table
  976773167          1         Sec GPT header
```

**SAVE THIS SOMEWHERE.**
You'll need this information to reconstruct the partition table.

Now, delete the GPT:

```bash
gpt destroy /dev/disk0
```

Create a new GPT:

```bash
gpt create -f /dev/disk0
```

Then, add all partitions one-by-one.
`-b` is the start block, `-s` is the size (number of blocks), `-i` is the index number (partition number), `-t` is the partition type.
You have to refer to the output from `gpt show` for the right values.
For the bad partition (with a bunch of Fs), the type you have to use is `53746F72-6167-11AA-AA11-00306543ECAC` for CoreStorage and `7C3457EF-0000-11AA-AA11-00306543ECAC` for APFS.

In my case, the commands I ran to re-add the partitions are:

```bash
gpt add -i 1 -b 40 -s 409600 -t C12A7328-F81F-11D2-BA4B-00A0C93EC93B disk0
gpt add -i 2 -b 409640 -s 878626472 -t 53746F72-6167-11AA-AA11-00306543ECAC disk0
gpt add -i 3 -b 879036112 -s 262144 -t 48465300-0000-11AA-AA11-00306543ECAC disk0
gpt add -i 4 -b 879560704 -s 91762688 -t 0FC63DAF-8483-4772-8E79-3D69D8477DE4 disk0
gpt add -i 5 -b 971323392 -s 1228799 -t C12A7328-F81F-11D2-BA4B-00A0C93EC93B disk0
gpt add -i 6 -b 972552192 -s 4220927 -t 0657FD6D-A4AB-43C4-84E5-0933C84B4F4F disk0
```

Then, verify the disk and its partitions:

```bash
diskutil list
diskutil verifyDisk disk0
diskutil verifyVolume disk0s1
diskutil verifyVolume disk0s2
diskutil verifyVolume disk0s3
diskutil verifyVolume disk0s4
diskutil verifyVolume disk0s5
diskutil verifyVolume disk0s6
```

Now you should be able to mount and read the disk.
