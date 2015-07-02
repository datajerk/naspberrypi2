# naspberrypi2
RPi 2 NAS Project

(images/naspberry.jpg)


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

## Static IP Address

I put a static IP address in my router to get this to stay the same IP address for all my servers. 

(images/static-routes.png)

## Configure RAID

```
ssh p
