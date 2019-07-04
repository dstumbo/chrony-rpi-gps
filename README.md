# chrony-rpi-gps
Chrony+gpsd ntp server on raspian buster lite using Adafruit Ultimate GPS hat and Adafruit external antenna for NMEA time and pps

A. Apparently in raspian buster, when gpsd uses ttyAMA0 (at least) it assumes the pps device. No need to modify config to add it (just use the -n parameter to make sure gpsd fires up right away). In fact, when I added /ddev/pps0 to the config file the pps lines were doubled in the gpsmon output.  
B. Apparently chrony for raspian buster includes pps support. Enabling it there works after apt-get install chrony (e.g. without a compile). I could never make the passthrough from gpsd to work. Got it on first try with chrony.  

C. Good reference for gpsd/chrony: https://forristal.org/blog/2018/01/making-a-raspberry-pi-stratum-1-clock/  

D. Good reference for getting rpi ready (mostly serial port): http://www.unixwiz.net/techtips/raspberry-pi3-gps-time.html  Note that this reference uses gpsd + ntp (not chrony)

E. https://wiki.alpinelinux.org/wiki/Chrony_and_GPSD and http://robotsforroboticists.com/chrony-gps-for-time-synchronization/ are also helpful  

F. Apparently kernel has or installs pps module automagically. No need to add the pps module manually.

G. Need to determine if "-r" in gpsd config helps or hurts. Need to determine what to do with hw_clock (disable fake-hwclock?)

Steps for headless Raspian Buster Lite on RPI 3B+:

0. Remove trace between pps pads on the Ultimate GPS hat. Solder a 560 ohm (or more, I'd use 1k if I had one handy but tested with 560) resistor across the pps pads (this might prevent blowing things up when both the GPS and RPI try to drive GPIO4. Put RTC battery (any CR12xx) in GPS hat. The RTC is not directly accessible, but the GPS uses it to speed up warm starts etc. Do NOT plug it into the RPI yet.

1. use balena etcher to write buster lite to SD card
2. touch ssh in /boot
3. copy wpa_supplicant.conf file into /boot (https://www.raspberrypi-spy.co.uk/2017/04/manually-setting-up-pi-wifi-using-wpa_supplicant-conf/). Edit with your wifi SSID and password. You can use: wpa_passphrase YOUR_SSID YOUR_PASSWORD to generate the hashed password instead of putting it in the file in cleartext.
4. add dtoverlay=pi3-disable-bt at the end of /boot/config.txt
5. turn off audio #dtparam=audio=on# in /boot/config.txt
6. Add dtoverlay=pps-gpio,gpiopin=4 to the end of /boot/config.txt (or gpiopin=18 if so wired)
7. put SD card in RPI and boot it
8. login over ssh: it will be ssh pi@raspberrypi.local if no name conflicts, otherwise find using newest dhcp lease
9. sudo apt-get update/upgrade
10. sudo raspi-config:  
	a. set hostname  
	b. local to en_US.UTF-8 UTF-8  
	c. set TZ to UTC  
	d. turn off serial port login shell, enable serial hardware  
	e. resize to fill SD card  
	f. minimize display memory to 16M since we have no display  
	g. reboot	
11. sudo systemctl disable hciuart
12. sudo apt-get install pps-tools, gpsd, gpsd-client, chrony
13. remove ntp-servers from /etc/dhcp/dhclient.conf
14. sudo rm /etc/dhcp/dhclient-exit-hooks.d/timesyncd
15. sudo rm /lib/dhcpcd/dhcpcd-hooks/50-ntp.conf 
16. sudo rm /var/lib/ntp/ntp.conf.dhcp (might not exist)
?? edit gpsd conf ?
?? edit chrony config ?
17. Make sure your antenna is outside (really!), with good sky view. This will greatly speed up/make possible acquisition of a fix good enough for pps. I never saw the WAAS sats with the antenna indoors on a south-facing windowsill, and my pps came and went randomly.
18. shutdown, power off, INSTALL GPS HAT, power on
19. using this approach, NMEA output will be on ttyAMA0. run gpsmon to see it (like this).  
20. Verify that GPS gets a 4D fix and goes to quality 2 (meaning it is using WAAS satellites). This may take 30 or more minutes, especially the first time before the RTC is correct.
21. after GPS hat LED flashing drops to once every 15 sec, verify pps: sudo ppstest /dev/pps0
22. Test chrony: run chronyc sources -v. After a few minutes, running it should produce a display like this:
23. Test chrony: run chronyc sourcestats -v. After a few minutes, running it should produce a display like this:
24. From another (linux or OS X) machine, test from the command line: ntpdate -q rpiname.local
25. Set your computer to use the rpi as its NTP server. Check that it works.  (out of scope) 
