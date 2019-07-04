# chrony-rpi-gps
Chrony+gpsd ntp server on raspian buster lite using Adafruit Ultimate GPS hat for NMEA time and pps

A. Apparently in raspian buster, when gpsd uses ttyAMA0 (at least) it assumes the pps device. No need to modify config to add it (just use the -n parameter). In fact, when I add /ddev/pps0 to the config file the pps lines are doubled in the gpsmon output.  
B. Apparently chrony for raspian buster includes pps support. Enabling it there works after apt-get install chrony (e.g. without a compile). I could never make the passthrough from gpsd to work. Got it on first try with chrony.  

C. Good reference for gpsd/chrony: https://forristal.org/blog/2018/01/making-a-raspberry-pi-stratum-1-clock/  

D. Good reference for getting rpi ready (mostly serial port): http://www.unixwiz.net/techtips/raspberry-pi3-gps-time.html  

E. https://wiki.alpinelinux.org/wiki/Chrony_and_GPSD and http://robotsforroboticists.com/chrony-gps-for-time-synchronization/ are also helpful  

F. Apparently kernel has or installs pps module automagically. No need to add the pps module manually.
