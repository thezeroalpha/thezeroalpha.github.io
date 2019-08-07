---
layout: post
title: "Test a Live USB With VirtualBox"
author: zeroalpha
categories: Guide
date: 2019-08-07
---

Often, I've wanted to test a live USB without having to reboot my computer.
This is made quite easy with VirtualBox, and here's how you do it.

In this guide, I will be explaining how to do it on macOS, but the procedure on Linux should be quite similar.
The `diskutil` command is Mac-specific; on Linux you can use `lsblk` to list disks/volumes, and `umount` to unmount disks/volumes.

First, plug the USB drive into your computer.
Then, figure out which disk and partition you want to live-boot.
To do this, run the following command:

```bash
diskutil list
```

In the output, you'll see a bunch of disks with volumes.
Find the volume that you want, based on e.g. the name, size, or mountpoint.
It should be under a title such as `/dev/disk3`, and it should have a number next to it (this refers to the partition number).

Here is what the output from `diskutil list` might look like (on Linux, `lsblk` should be similar, though you may have to set output columns with `lsblk -o`):

![Sample output from diskutil list](/img/diskutil-list-output.jpg)

For this example, let's take partition 1 of `/dev/disk3`.
This is referred to as `/dev/disk3s1`.

Make sure the disk is unmounted by running:

```bash
diskutil unmount /dev/disk3s1
```

(of course, replacing `/dev/disk3s1` with the disk you selected).
On Linux, the same can generally be achieved with `sudo umount /dev/disk3s1`.

Then, create the virtual machine disk, using VBoxManage:

```bash
sudo VBoxManage internalcommands createrawvmdk -filename ~/live.vmdk -rawdisk /dev/disk3s1
```

This will create a raw virtual machine disk at the location you specify, here we choose `~/live.vmdk` (but you can adjust this to your preference).
This path should also be substituted in any other commands.
The disk we pass as the argument is the running example of `/dev/disk3s1`, but this should be the disk you chose above.

Make sure not to disconnect your actual USB disk from the computer, as the raw disk file will still be using your device.

Next, for VirtualBox to be able to access the USB, you have to set permissions accordingly.

Make yourself the owner of the disk you created:

```bash
sudo chown `whoami` ~/live.vmdk
```

Set the required permissions:

```bash
chmod 777 ~/live.vmdk
```

You also need to set these permissions on the actual USB device (substituting the `/dev` path with the one you selected):

```bash
sudo chmod 777 /dev/disk3s1
```

Open the VirtualBox GUI, and create a new machine.
Set up the parameters (machine folder, type, version, memory size) however you like, just make sure to choose a 64-bit version if you're booting a 64-bit system from the USB, and 32-bit for 32-bit systems.

When you get to the hard disk selection menu, choose the button labeled "use an existing virtual hard disk file", then click the folder icon next to the menu, click "add" at the top, and select the `.vmdk` file you created.

Now everything should be set up.
You can click "start" in VirtualBox, and your machine should start up from the USB disk.
