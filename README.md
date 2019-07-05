# chrony-rpi-gps
Chrony+gpsd ntp server on raspbian buster lite using Adafruit Ultimate GPS hat (product 2324) and Adafruit active external antenna (products 960 & 851) for NMEA time and pps

  
A. Apparently chrony for raspbian buster includes pps support. It works after simple apt-get install chrony (e.g. without a compile). I could never make the passthrough from gpsd to work. Got it on first try with chrony.  

B. Good reference for gpsd/chrony: https://forristal.org/blog/2018/01/making-a-raspberry-pi-stratum-1-clock/  

C. Good reference for getting rpi ready (mostly serial port): http://www.unixwiz.net/techtips/raspberry-pi3-gps-time.html  Note that this reference uses gpsd + ntp (not chrony)

D. https://wiki.alpinelinux.org/wiki/Chrony_and_GPSD and http://robotsforroboticists.com/chrony-gps-for-time-synchronization/ are also helpful  

E. Apparently kernel has or installs pps module automagically. No need to add the pps module manually.

F. install procedure (below) tested on RPI 3B+ (works). Resulting card tested on RPI 3B (works), and RPI zero WH (works).

Steps for headless Raspbian Buster Lite on RPI 3B+:

0. Remove trace between pps pads on the Ultimate GPS hat. Solder a 560 ohm (or more, I'd use 1k if I had one handy but tested with 560) resistor across the pps pads (this might prevent blowing things up when both the GPS and RPI try to drive GPIO4. Put RTC battery (any CR12xx) in GPS hat. The RTC is not directly accessible, but the GPS uses it to speed up warm starts etc. Do NOT plug it into the RPI yet.

1. Use balena etcher to write buster lite to SD card (4G or larger)
2. Touch ssh in /boot (e.g. in ubuntu touch /media/username/boot/ssh)
3. Copy wpa_supplicant.conf file (see example file) into /boot (https://www.raspberrypi-spy.co.uk/2017/04/manually-setting-up-pi-wifi-using-wpa_supplicant-conf/). Edit with your wifi SSID and password. You can use: wpa_passphrase YOUR_SSID YOUR_PASSWORD to generate the hashed password instead of putting it in the file in cleartext.
4. Add dtoverlay=pi3-disable-bt at the end of /boot/config.txt
5. Turn off audio #dtparam=audio=on# in /boot/config.txt
6. Add dtoverlay=pps-gpio,gpiopin=4 to the end of /boot/config.txt (or gpiopin=18 if so wired)
7. Put SD card in RPI and boot it
8. Login over ssh: it will be ssh pi@raspberrypi.local if no name conflicts, otherwise find using newest dhcp lease
9. sudo apt-get update/upgrade
10. sudo raspi-config:  
	a. set hostname  
	b. change locale to en_US.UTF-8 UTF-8  
	c. set TZ to UTC  
	d. turn off serial port login shell, enable serial hardware  
	e. change password if desired  
	f. minimize display memory to 16M since we have no display  
	g. resize to fill SD card  
	h. reboot	
11. sudo systemctl disable hciuart
12. sudo apt-get install pps-tools gpsd gpsd-clients chrony
13. Remove ntp-servers from /etc/dhcp/dhclient.conf
14. sudo rm /etc/dhcp/dhclient-exit-hooks.d/timesyncd
15. sudo rm /lib/dhcpcd/dhcpcd-hooks/50-ntp.conf (might not exist)
16. sudo rm /var/lib/ntp/ntp.conf.dhcp (might not exist)  
17. Edit /etc/default/gpsd to add turn off usbauto, add serial device and option -n (see example file)  
18. Edit /etc/chrony/chrony.conf to choose ntp pool, configure initial slewing and add the GPS and PPS sources (see example file)
19. Make sure your antenna is outside (really!), with good sky view. This will greatly speed up/make possible acquisition of a fix good enough for pps. I never saw the WAAS sats with the antenna indoors on a south-facing windowsill, and my pps came and went randomly.
20. shutdown, power off, INSTALL GPS HAT, power on
21. Using this approach, NMEA output will be on ttyAMA0. run gpsmon to see it (see example gpsmon output file)  
22. Verify that GPS gets a 4D fix and goes to quality 2 (meaning it is using WAAS satellites). This may take 30 or more minutes, especially the first time before the RTC is correct.
23. after GPS hat LED flashing drops to once every 15 sec, verify pps: sudo ppstest /dev/pps0. Should get a new line every second, with incrementing sequence numbers
24. Test chrony: run chronyc sources -v. After a few minutes, you should get something like the example sources output file
25. Test chrony: run chronyc sourcestats -v. After a few minutes, you should get something like the example sourcestats output file
26. From another (linux or OS X) machine, test from the command line: ntpdate -q rpiname.local
27. Set your computer to use the rpi as its NTP server. Check that it works.  (out of scope) 
