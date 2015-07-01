# naspberrypi2
RPi 2 NAS Project


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

