## Overview

Setting up for NAS servers using two Raspberry Pi 4, 4Gb devices.

I have *alot* of data backed up into the cloud but I'm not 100% sure that I could ever get it all back!  Also I have been unfortunate enough to have fairly slow internet connections at home so even if I trust the cloud backup, restoring from that is slow.

The solution (hopefully) is a cheap (software) RAID system using Raspberry Pi's for the computing and cheap (relatively), 2.5", external, USB drives for storage.

## Hardware

### Raspberry Pi 4

The recently upgraded version of the [Raspberry Pi](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/)

I'm using the 4Gb versions here, available from the following

* [Pimoroni](https://shop.pimoroni.com/products/raspberry-pi-4?variant=29157087445075)
* [The Pi Hut](https://thepihut.com/products/raspberry-pi-4-model-b?variant=20064052740158)
* [CPC Farnell](https://cpc.farnell.com/raspberry-pi/rpi4-modbp-4gb/raspberry-pi-4-model-b-4gb/dp/SC15185)


### Powered USB Hub

I actually have a few of these hubs and they seem to work fairly well.  I mainly started to use them since the older Raspberry Pi (and other devices I have) only come with USB2.0 ports and these often can't provide enough power for external hard drives.

* [AUKEY 7 port USB Hub](https://www.amazon.co.uk/AUKEY-SuperSpeed-Adapter-Transfer-Windows/dp/B01NAS2639/ref=sr_1_3?keywords=AUKEY+USB+Hub+7&qid=1566989723&s=computers&sr=1-3)

Any powered hub is probably going to be fine.  This one is 30W or 6A at 5V.

I couldn't find the power requirements for the USB drives but this seems like a reasonable guide to the [drives themselves](https://www.seagate.com/www-content/datasheets/pdfs/barracuda-2-5-DS1907-2-1907US-en_GB.pdf)

The drives in the USB enclosures are likely to be 5400rpm so lower spec than this.  Max current for 4 drives @1.2 each is 4.8A which leaves some headroom.


### Hard drives

4Tb Seagate external 2.5" USB3.0 drives.


## Initial Drive Installation

### Labelling

I wanted to know which drive was which so that, in the case of a failure, I can pick the right one to replace.  It seems from previous experience that Linux scans devices on the hub in a certain order.  It starts with the port nearest the green power light on the hub and works along the devices.

Using this approach I was able to plug in and label the drives as "sda", "sdb", etc.


### Remove partitions

Run the following command as root

```
fdisk /dev/sda
```

The following shows the commands needed to delete the partitions (two on these drives)

```
Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): d
Partition number (1,2, default 2): 

Partition 2 has been deleted.

Command (m for help): d
Selected partition 1
Partition 1 has been deleted.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

### Install mdadm

This is the RAID configuration tool for Linux.

For Debian Buster it seems to have dependencies on [MTA](https://packages.debian.org/buster/mdadm)



### Create Array

NOTE: This takes a LONG time.  Given the size of the array and the speed restrictions on the Raspberry Pi's IO this is not a fast process.  

For the 3 disc RAID 5 array it took around 24-36hours to fully create the array.

For the 4 disc RAID 6 array it was more like 4 days.


#### 3 disc RAID 5

Running the following command

```
mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sda /dev/sdb /dev/sdc
```

Gives the following output

```
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: partition table exists on /dev/sda
mdadm: partition table exists on /dev/sda but will be lost or
       meaningless after creating array
mdadm: partition table exists on /dev/sdb
mdadm: partition table exists on /dev/sdb but will be lost or
       meaningless after creating array
mdadm: partition table exists on /dev/sdc
mdadm: partition table exists on /dev/sdc but will be lost or
       meaningless after creating array
mdadm: size set to 3906886144K
mdadm: automatically enabling write-intent bitmap on large array
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

Checking on the progress of the array building

```
cat /proc/mdstat 
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sdc[3] sdb[1] sda[0]
      7813772288 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/2] [UU_]
      [==>..................]  recovery = 10.8% (424562304/3906886144) finish=850.6min speed=68229K/sec
      bitmap: 2/30 pages [8KB], 65536KB chunk

unused devices: <none>
```

Apparently the creation of the array will use the "recovery process" but it's still usable at this point (see [Digital Ocean RAID Tutorial](https://www.digitalocean.com/community/tutorials/how-to-create-raid-arrays-with-mdadm-on-debian-9)



#### 4 disc RAID 6

Run the following command to create the array

```
mdadm --create --verbose /dev/md0 --level=6 --raid-devices=4 /dev/sda /dev/sdb /dev/sdc /dev/sdd
```

Gives the following output

```
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: partition table exists on /dev/sda
mdadm: partition table exists on /dev/sda but will be lost or
       meaningless after creating array
mdadm: partition table exists on /dev/sdb
mdadm: partition table exists on /dev/sdb but will be lost or
       meaningless after creating array
mdadm: partition table exists on /dev/sdc
mdadm: partition table exists on /dev/sdc but will be lost or
       meaningless after creating array
mdadm: partition table exists on /dev/sdd
mdadm: partition table exists on /dev/sdd but will be lost or
       meaningless after creating array
mdadm: size set to 3906886144K
mdadm: automatically enabling write-intent bitmap on large array
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

Check on the progress of the building of the array

```
cat /proc/mdstat 
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid6 sdd[3] sdc[2] sdb[1] sda[0]
      7813772288 blocks super 1.2 level 6, 512k chunk, algorithm 2 [4/4] [UUUU]
      [>....................]  resync =  0.2% (7861440/3906886144) finish=1495.7min speed=43443K/sec
      bitmap: 30/30 pages [120KB], 65536KB chunk

unused devices: <none>
```

Create a filesystem on the array

```
mkfs.ext4 -F /dev/md0
```

Add an entry for fstab

```
echo '/dev/md0 /mnt/md0 ext4 defaults,nofail,discard 0 0' | tee -a /etc/fstab
```

## References 

* [Digital Ocean RAID Tutorial](https://www.digitalocean.com/community/tutorials/how-to-create-raid-arrays-with-mdadm-on-debian-9)
