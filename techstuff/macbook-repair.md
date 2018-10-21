---
title: Macbook Repair
---

# Macbook Troubleshooting Steps
1. Reset SMC — shut down, plug in charger, hold shift+ctrl+alt+power and let go of all at once. charger light should change color.
2. PRAM/NVRAM — shut down, turn on, hold cmd+alt+P+R on start up sound, wait until next start up sound, let go
3. Do steps 1 and 2 again.
4. Disk repair
    * Live mode Disk Utility — open disk utility, run First Aid
    * Recovery mode Disk Utility — shut down, turn on and hold cmd+r, go into disk utility and run First Aid
    * fsck (file system check) — shut down, turn on and hold cmd+s, type /sbin/fsck -fy, run over and over until no errors
5. Apple Hardware Test — shut down, turn on and hold D, run the test
6. Cry. And probably `testdisk` to at least save some data.
