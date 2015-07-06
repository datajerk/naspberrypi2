# naspberrypi2

RPi 2 NAS Project

This is instructions on how to make a home RAID 1 NAS using a Raspberry Pi and two 3TB hard drives.  
This home NAS can be used to store backups of pictures, media, etc, in our house. 
The performance will not knock your socks off.  But its good enough, and its for fun. 

![alt setup](images/naspberry.jpg)


## Shopping List

* [Raspberry Pi](http://www.amazon.com/gp/product/B00TFV5QTA)
* [2 x 3TB USB Drives](http://www.amazon.com/gp/product/B00E3RH63A)
* [1 x 16GB SD Card](http://www.amazon.com/gp/product/B00M55C0LK)

Total price for me on June 26, 2015 was $243.11

## Install OS

I used torent.  Downloaded from: 
http://www.raspberrypi.org/downloads/

Create SD Card to boot from.  Instructions are here: 
https://www.raspberrypi.org/documentation/installation/installing-images/mac.md

```
unzip 2015-05-05-raspbian-wheezy.zip
Archive:  2015-05-05-raspbian-wheezy.zip
  inflating: 2015-05-05-raspbian-wheezy.img
```
This expands a 3.1GB file.  Make sure you have room!  

Now we put an SD card into the Mac.  From there we look at the output of diskutil -l 
and we can see that our SD card is #3.  Yours may be different.  

Copy the image into the SD card with the below commands:

```
diskutil unmountDisk /dev/disk3
sudo dd bs=1m if=2015-05-05-raspbian-wheezy.img of=/dev/disk3
```

This operation takes about 18 min on my machine. 

Put the SD card into the Raspberry Pi, turn it on and do basic config.  Log in remotely.

## Config OS

``` 
sudo -s
apt-get update && apt-get dist-upgrade -y && rpi-update && reboot
```

Swap to 2GB (remember this old rule that swap is double of memory?). 
We configure swap to be a file on the RAID1 partition we are going to create.  Swapping
on the SD card is bad.
```
cat /etc/dphys-swapfile
CONF_SWAPSIZE=2048
CONF_SWAPFILE=/vol1/dswap
```

And since I always use vim instead of vi by habit: 
```
apt-get install -y vim
```

Also, copy my keys for SSH.  On my mac run: 
```
scp ~/.ssh/id_rsa.pub pi@192.168.128.214:/home/pi/.ssh/authorized_keys
```
Now I can ssh without password. 


## Static IP Address

I put a static IP address in my router to get this to stay the same IP address for all my servers. 

![alt router](images/static-route.png)

## Configure Filesystem and RAID

```
ssh pi@192.168.128.214
sudo apt-get install mdadm 
```
When you get a prompt asking to type something, press 'Ok', then type 'none'

```
sudo -s
fdisk /dev/sda
d (this will delete the current /dev/sda1 if you have one, if not skip)
n
enter
enter
enter
enter
t
fd
w
```
Repeat above for /dev/sdb

```
mdadm --zero-superblock /dev/sda1
mdadm --zero-superblock /dev/sdb1
```
Now create the new RAID: 
```
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1
```
Make sure you see the md0 device: 
```
cat /proc/partitions
```
watch it build: 
```
watch cat /proc/mdstat
```
This really does take a long time... do something else while this is happening.  

When its done, come back and make a filesystem! 
```
mkfs.ext4 /dev/md0
When its done, come back and make a filesystem! 
```
That takes another 30 min or so... 

Now mount it. 

```
mkdir /vol1
mount /dev/md0 /vol1
```

### Make it persistent
```
mdadm --detail --scan >>/etc/mdadm/mdadm.conf
```
Now edit /etc/fstab and add the line: 
```
/dev/md0        /vol1           auto    defaults        0       0
```

Edit /boot/cmdline and make sure it comes back up by adding: 
```
rootdelay=5
```
To the line.  Mine looks like this: 
```
dwc_otg.lpm_enable=0 console=ttyAMA0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline rootdelay=5 rootwait
```
Now, Reboot to make sure it comes back up. 

```
shutdown -r now
```

When it boots up, you should be able to log in and see the volumes persist. 

## Time Machine and Apple Devices

```
sudo apt-get -y install netatalk
sudo update-rc.d netatalk defaults
```

Configure drives (make sure you make this with the pi user who will be accessing this)
```
sudo chmod 777 /vol1
mkdir /vol1/Photos
mkdir /vol1/Media
mkdir /vol1/TimeMachine-iMac
mkdir /vol1/TimeMachine-MBP
```
Create config file.  As root, edit /etc/netatalk/AppleVolumes.default.  Add these lines: 
```
/vol1/Media             "iTunes Media"
/vol1/TimeMachine options:tm        "All TimeMachine Disks"
/vol1/Photos            "Photos"
```
That gives me two TimeMachine capsules, photos and Media folders that will share.  Restart netatalk and mount it 
to your iDevices!
```
service netatalk restart
```

You should now be able to see them on the net with your Apple devices.  Happy backing up!

## Sources

* http://www.mikronauts.com/raspberry-pi/raspberry-pi-2-nas-experiment-howto/
* http://www.mikronauts.com/raspberry-pi/raspberry-pi-2-usb-hard-drive-and-adapter-tests/ 
* https://www.raspberrypi.org/forums/viewtopic.php?t=38429 
* http://www.davidhunt.ie/raid-pi-raspberry-pi-as-a-raid-file-server/
* https://www.raspberrypi.org/forums/viewtopic.php?f=66&t=99447 
* https://www.debian.org/doc/manuals/debian-reference/ch09.en.html#_data_encryption_tips 
* https://thelastmaimou.wordpress.com/2014/04/07/picryption-truecrypt-for-the-pi-2/
* https://raymii.org/s/articles/Build_a_35_dollar_Time_Capsule_-_Raspberry_Pi_Time_Machine.html
* https://www.davidschlachter.com/misc/netatalk3rpi
* https://www.raspberrypi.org/forums/viewtopic.php?f=36&t=84187
* https://kremalicious.com/ubuntu-as-mac-file-server-and-time-machine-volume/ 
