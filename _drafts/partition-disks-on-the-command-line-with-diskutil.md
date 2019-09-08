---
layout: post
title: "Partition Disks on the Command Line with diskutil"
author: zeroalpha
categories: Guide
---

![Why Disk Utility sucks](/img/disk-utility-cannot-partition.png)

List disks:

```bash
diskutil list
```

List a disk and its volumes:

```bash
diskutil list /dev/disk4
```

Output:

```
/dev/disk4 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *62.0 GB    disk4
   1:                 DOS_FAT_32 PHOTOS                  62.0 GB    disk4s1
```


Choose a filesystem:

```bash
diskutil listFilesystems
```

Output:

```
Formattable file systems

These file system personalities can be used for erasing and partitioning.
When specifying a personality as a parameter to a verb, case is not considered.
Certain common aliases (also case-insensitive) are listed below as well.

-------------------------------------------------------------------------------
PERSONALITY                     USER VISIBLE NAME
-------------------------------------------------------------------------------
APFS                            APFS
  (or) APFSI
Case-sensitive APFS             APFS (Case-sensitive)
ExFAT                           ExFAT
Free Space                      Free Space
  (or) FREE
MS-DOS                          MS-DOS (FAT)
MS-DOS FAT12                    MS-DOS (FAT12)
MS-DOS FAT16                    MS-DOS (FAT16)
MS-DOS FAT32                    MS-DOS (FAT32)
  (or) FAT32
HFS+                            Mac OS Extended
Case-sensitive HFS+             Mac OS Extended (Case-sensitive)
  (or) HFSX
Case-sensitive Journaled HFS+   Mac OS Extended (Case-sensitive, Journaled)
  (or) JHFSX
Journaled HFS+                  Mac OS Extended (Journaled)
  (or) JHFS+
```

Erase a disk:

```bash
diskutil eraseDisk FAT32 PHOTOS /dev/disk4
```

Delete contents of a disk without needing the full command:

```bash
diskutil reformat /dev/disk4
```

Partition a disk:

```bash
diskutil partitionDisk /dev/disk4 MBR FAT32 SYSRESC 4g FAT32 MUSIC 10g FAT32 PHOTOS 0b
```

Output:

```
/dev/disk4 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *62.0 GB    disk4
   1:                 DOS_FAT_32 SYSRESC                 4.0 GB     disk4s1
   2:                 DOS_FAT_32 MUSIC                   10.0 GB    disk4s2
   3:                 DOS_FAT_32 PHOTOS                  48.0 GB    disk4s3
```


