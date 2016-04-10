# RPI 3 Image

### Get the image on the Pi

This wiki is aimed at helping you get the RADIUSDesk RPI image from SourceForge, onto your very own Pi.

1. Download the [Operating system image.](https://sourceforge.net/projects/radiusdesk/files/RaspberryPi/)
2. Extract the image.
  * The image is in a tar file, which is then compressed using 7Zip.
  * you will need to extract this with what ever method you find best. A quick google search will aid you. You could use the 7Zip package [here](http://www.7-zip.org/download.html).
  * Now you should have a *.img* file, which you can now write to an sdcard.
3. Write the image to a Micro sdcard of your choosing.
  * You will notice the image is not that big. So you should be able to use a fairly small card. I recommend 8 Gig though.
  * Using `dd`,the *Restore Disk image* facility in Ubuntu, *Disks*, or another method in windows. The Raspberry Pi foundation have created an awesome guide for this already. [Link](https://www.raspberrypi.org/documentation/installation/installing-images/)
4.  
