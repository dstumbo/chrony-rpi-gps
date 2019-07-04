# chrony-rpi-gps
Chrony+gpsd ntp server on raspian buster lite using Adafruit Ultimate GPS hat for NMEA time and pps

A. Apparently in raspian buster, when gpsd uses ttyAMA0 (at least) it assumes the pps device. No need to modify config to add it (just use the -n parameter). In fact, when I add /ddev/pps0 to the config file the pps lines are doubled in the gpsmon output.  
B. Apparently chrony for raspian buster includes pps support. Enabling it there works after apt-get install chrony (e.g. without a compile). I could never make the passthrough from gpsd to work. Got it on first try with chrony.  

C. Good reference for gpsd/chrony: https://forristal.org/blog/2018/01/making-a-raspberry-pi-stratum-1-clock/  

D. Good reference for getting rpi ready (mostly serial port): http://www.unixwiz.net/techtips/raspberry-pi3-gps-time.html  

E. https://wiki.alpinelinux.org/wiki/Chrony_and_GPSD and http://robotsforroboticists.com/chrony-gps-for-time-synchronization/ are also helpful  

F. Apparently kernel has or installs pps module automagically. No need to add the pps module manually.

Steps for Raspian Buster Lite on RPI 3B+:

0. remove trace between pps pads on Ultimate GPS. Solder 560 ohm (or more, I'd use 1k if I had one but I tested with 560) resistor across pps pads (this might prevent blowing things up when both the GPS and RPI try to drive GPIO4. put rtc battery (any CR12xx) in gps hat. The RTC is not directly accessible, but the gps uses it to do warm starts etc. Need to determine if "-r" helps or hurts. Need to determine what to do with hw_clock.

1. use balena etcher to write buster lite to SD card
2. touch ssh on boot
3. copy wpa_supplicant.conf file onto boot (https://www.raspberrypi-spy.co.uk/2017/04/manually-setting-up-pi-wifi-using-wpa_supplicant-conf/)
4. boot
5. login over ssh (find using newest dhcp lease?) will be ssh pi@raspberrypi.local if no name conflicts
6. sudo apt-get update/upgrade
7. sudo raspi-config:
	a. set hostname	
	b. local to en_US.UTF-8 UTF-8
	c. set TZ to UTC
	d. turn off serial port login shell, enable hardware
	e. resize
	f. minimize display memory to 16M since we have no display
	g. reboot	
8. insert dtoverlay=pi3-disable-bt in /boot/config.txt
9. turn off audio #dtparam=audio=on#
10. Add dtoverlay=pps-gpio,gpiopin=4 to the end of /boot/config.txt (or gpiopin=18 if so wired)
11. sudo systemctl disable hciuart
12. sudo apt-get install pps-tools, gpsd, gpsd-client, chrony
13. remove ntp-servers from /etc/dhcp/dhclient.conf
14. sudo rm /etc/dhcp/dhclient-exit-hooks.d/timesyncd
15. sudo rm /lib/dhcpcd/dhcpcd-hooks/50-ntp.conf 
??. sudo rm /var/lib/ntp/ntp.conf.dhcp (might not exist)
16. Make sure your antenna is outside, with good sky view. This will greatly speed up acquisition of a fix good enough for pps. I never saw the WAAS sats with the antenna indoors on a south-facing windowsill.
12. reboot
13. test
??. Add pps-gpio to the end of /etc/modules unnecessary?)
12. shutdown, power off, INSTALL GPS HAT
13. using this approach, NMEA output seems to be on /dev/serial0 (ttyAMA0 has something similar, but with garbage??). Have to use AMA0, seems like gpsd is hardcoded to associated SOCK for chrony
14. apt-get install pps-tools and after flashing drops to 1/15 sec, verify
15. Disable NTP support in DHCP and install GPSD as in http://www.unixwiz.net/techtips/raspberry-pi3-gps-time.html
??. add device tree overlay for RTC (probably not), disable fake-hwclock
?? Something is talking to GPIO4: could it be chrony trying to talk to an RTC via 1-wire
?? add -r in gpsd conf to make it report internal RTC when GPS isn't ready - anything is better tha rpi hwclock
