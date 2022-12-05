# How to attach external HDD to Raspberry Pi 4

## OS Configurations
```bash
cat /etc/os-release

PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

```text
uname --all

Linux cranberrypi 5.15.76-v8+ #1597 SMP PREEMPT Fri Nov 4 12:16:41 GMT 2022 aarch64 GNU/Linux
```
## Can Raspberry Pi can see the hard disk during boot process?

Using `dmesg` command

```bash
dmesg | tail -n 20

[   98.521088] usb 2-2: new SuperSpeed USB device number 2 using xhci_hcd
[   98.544671] usb 2-2: New USB device found, idVendor=1f75, idProduct=0611, bcdDevice= 0.06
[   98.544757] usb 2-2: New USB device strings: Mfr=4, Product=5, SerialNumber=6
[   98.544776] usb 2-2: Product: Ext. HDD
[   98.544790] usb 2-2: Manufacturer: Innostor
[   98.544804] usb 2-2: SerialNumber: 20220616
[   98.549981] usb-storage 2-2:1.0: USB Mass Storage device detected
[   98.551832] scsi host0: usb-storage 2-2:1.0
[   99.553607] scsi host0: scsi scan: INQUIRY result too short (5), using 36
[   99.553627] scsi 0:0:0:0: Direct-Access     WDC WD10 SPZX-22Z10T1          PQ: 0 ANSI: 0
[   99.554179] sd 0:0:0:0: [sda] 1953525168 512-byte logical blocks: (1.00 TB/932 GiB)
[   99.555222] sd 0:0:0:0: [sda] Write Protect is off
[   99.555234] sd 0:0:0:0: [sda] Mode Sense: 3b 00 00 00
[   99.556286] sd 0:0:0:0: [sda] No Caching mode page found
[   99.556294] sd 0:0:0:0: [sda] Assuming drive cache: write through
[   99.570018] sd 0:0:0:0: Attached scsi generic sg0 type 0
[   99.642912] sd 0:0:0:0: [sda] Attached SCSI disk
```

We can see that Raspberry Pi can see or detect the `WDC WD10 SPZX-22Z10T1` hard disk. That is Western Digital WD Blue<sup>TM</sup> and you can see the hard disk's [specifications](https://documents.westerndigital.com/content/dam/doc-library/en_us/assets/public/western-digital/product/internal-drives/wd-blue-hdd/specification-sheet-wd-blue-mobile-sata-hdd.pdf).


## List the block devices

A block device is a storage device that we can access information in any order. Hard disks are examples of block device. To list all the block devices, use the `lsblk` command.

```bash
sudo lsblk -l

NAME      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda         8:0    0 931.5G  0 disk
mmcblk0   179:0    0  29.7G  0 disk
mmcblk0p1 179:1    0   256M  0 part /boot
mmcblk0p2 179:2    0  29.5G  0 part /
```
All the devices populate the `/dev` folder. In this case the block device `sda` will be at `/dev/sda`.

## Display or manipulate a disk partition table.

Using `fdisk` command, we can see the current partition table.

```bash
sudo fdisk -l /dev/sda

Disk /dev/sda: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: SPZX-22Z10T1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```
## Create a partition table

A partition table is the data in the boot sector of a hard disk that describes how it is divided into drives. The partition table is loaded and read to determine which partition contains the active operating system.

A partition is a reserved part of hard disk that is treated as a separate drive.

Using `fdisk` command, we can create a new partition table for our `/dev/sda` block device.

```bash
sudo fdisk /dev/sda

Welcome to fdisk (util-linux 2.36.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x51eb7570.

Command (m for help): g
```
Type `g` to create [GUID Partition Table](https://en.wikipedia.org/wiki/GUID_Partition_Table)

```bash
Created a new GPT disklabel (GUID: 1DAE17FD-AF0A-A640-BD7A-XXXXXXXXXXXX).
```

Type `n` to add a new partition, and type in `1` to create only one single partition and accept the default value by pressing enter.

```bash
Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-1953525134, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-1953525134, default 1953525134):

Created a new partition 1 of type 'Linux filesystem' and of size 931.5 GiB.
```

Type `w`, to write partition table to hard disk and exit

```bash
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Run `fdisk` command again to see the new partition.

```bash
sudo fdisk -l /dev/sda

Disk /dev/sda: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: SPZX-22Z10T1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 1DAE17FD-AF0A-A640-BD7A-XXXXXXXXXXXX

Device     Start        End    Sectors   Size Type
/dev/sda1   2048 1953525134 1953523087 931.5G Linux filesystem
```

## Format the partition

Using `mkfs`, we will create an [ext4](https://en.m.wikipedia.org/wiki/Ext4) Linux filesystem.

```bash
sudo mkfs.ext4 /dev/sda1

mke2fs 1.46.2 (28-Feb-2021)
Creating filesystem with 244190385 4k blocks and 61054976 inodes
Filesystem UUID: 30b1ea61-8b98-4919-bcf0-bd5a2c57d5e0
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
	102400000, 214990848

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done
```

## Mount the file system

Using `mount` command, we will mount the file system to `/media/wd10`, but first, we need to create a directory with `mkdir` command.

```bash
sudo mkdir /media/wd10
sudo mount /dev/sda1 /media/wd10
```

Check if the file system was mount successfully.

```bash
mount -l | grep sda1

/dev/sda1 on /media/wd10 type ext4 (rw,relatime)
```

## Automounting the file system

If we want to mount the file system when we boot the system, we need to edit `fstab` file.

```bash
sudo vim /etc/fstab
```
Add the following line

```text
/dev/sda1	/media/wd10	ext4	defaults	0	0
```
Save the file and exit.

```bash
cat /etc/fstab

proc            /proc           proc    defaults          0       0
PARTUUID=15470b2d-01  /boot           vfat    defaults          0       2
PARTUUID=15470b2d-02  /               ext4    defaults,noatime  0       1
/dev/sda1	/media/wd10	ext4	rw,user 	0	0
```

## Test the `fstap` file

```bash
sudo mount -a
```

## Unmount file system

Simply use `umount` command.

```bash
sudo umount /media/wd10
```

## Troubleshooting
### Slow reboot/boot check
```bash
systemd-analyze blame

systemd-analyze critical-chain
```