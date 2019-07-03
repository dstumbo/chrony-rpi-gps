# chrony-rpi-gps
chrony on rpi using Adafruit Ultimate GPS hat for NMEA time and pps

A. Apparently in buster, when gpsd uses ttyAMA0 (at least) it assumes the pps device. do not need to modify config to add it (just enable the -n parameter). In fact, when I add it to the command line I see it the pps lines doubled in the gpsmon output.  
B. Apparently chrony for raspian buster includes pps support. enabling it there works after just apt-get install chrony (without a compile). I could never make the passthrough from gpsd to work. Got it on first try with chrony.  

C. best reference for gpsd/chrony: https://forristal.org/blog/2018/01/making-a-raspberry-pi-stratum-1-clock/  

D. Best reference for getting rpi ready: http://www.unixwiz.net/techtips/raspberry-pi3-gps-time.html  

E. https://wiki.alpinelinux.org/wiki/Chrony_and_GPSD and http://robotsforroboticists.com/chrony-gps-for-time-synchronization/ are also helpful  

F. Apparently kernel has or installs pps module automagically. 
