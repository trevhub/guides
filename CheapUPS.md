# Protecting your Rocketpool node from power outages on the cheap #

An uninterruptible power supply (UPS) is a device that allows a computer to keep running for at least a short time when incoming power is interrupted. Along with the obvious power connection, a secondary connection via USB can trigger a **graceful shutdown** to your staking machine in the event of a power loss. Without this in place, a **hard shutdown** can cause corruption of program and operating system files causing the need to hop on the rescue node and re-sync from scratch. With Ice storms, hurricanes, and terrible power utility infrastructure I wanted to protect against these events the cheapest way possible. My goal wasn’t to stay up with battery power for any meaningful length of time. I just wanted to gracefully shut down my staking machine, then to automatically start up and continue normal operation when power was restored. Total hands-off, worry free operation. 

Most staking machines use under 25W of power so the load requirements are pretty low. I looked for the cheapest UPS **with USB connectivity** needed for a graceful shutdown command to be sent to a PC. There are less expensive UPS options but they lack this function.

## Cyberpower CP425SLG ## 
### MSRP: $57.95USD ###
![CP425SLG](/../photos/CP425SLG_F2.jpg)

Specs:
- Capacity: 425 VA / 255 W
- Topology: Standby
- Waveform: Simulated Sine Wave
- Output: 120 VAC ± 5%
- Plug type & cord: NEMA 5-15P, 5 ft. cord
- Outlet types: 8 × NEMA 5-15R
- Communication ports: USB
- Management software: PowerPanel® Personal
- Warranty: 3 year
- Connected Equipment Guarantee: $75,000

A 255W capacity is more than enough for my needs.
```
staking machine (13W) 
fiber modem (25W) 
router (10W) 
Total Load = 48W
```
48W works out to 19% load on the CP425SLG and will give me 36 minutes of uptime. ([or 8% Load 3h 13 minutes on the $599 APC SMC1500C](https://www.apc.com/us/en/product/SMC1500C/apc-smartups-c-line-interactive-1440va-tower-120v-8x-nema-515r-outlets-smartconnect-port-usb-and-serial-communication-avr-graphic-lcd/))

The connections:
![ups connections](/../photos/UPSconnections.png)
The left side of the CP425SLG is for *surge protection only* and doesnt run on battery power during an outage, I have my TV, Stereo, Laptop plugged in to those outlets. 

## **Installing the Cyberpower software on your node:** ##

My options were **[Cyberpowers PowerPanel](https://www.cyberpowersystems.com/product/software/power-panel-personal/powerpanel-for-linux/)** to configure and monitor my UPS or **[Network UPS Tools](https://networkupstools.org/)**. NUT is A free and open source UPS program licensed under the terms of GNU Public License (GPL). NUT works with 170 different manufacturers, and several thousands of models. I went with PowerPanel.

### *Security Note: You are about to install software on your staking machine, always proceed with caution and verify URLs.* ###

I went with Cyberpowers Powerpanel.
https://www.cyberpowersystems.com/product/software/power-panel-personal/powerpanel-for-linux/

choose your install file.
![powerpanel](/../photos/powerpanel.png)

Log into your staking machine and download the `*deb` file replacing `[CORRECT URL]` with the address

`curl -O https://[CORRECT URL]/PPL_64bit_v1.4.1.deb`

Install this package by replacing [PATH] with the location of the downloaded .deb file.

`sudo dpkg -i [PATH]/PPL_64bit_v1.4.1.deb`

connect the USB cable from UPS to your node. 

`sudo pwrstat -config` will show your current UPS config.
```
Daemon Configuration:

Alarm .............................................. On
Hibernate .......................................... Off
Cloud .............................................. Off

Action for Power Failure:

	Delay time since Power failure ............. 60 sec.
	Run script command ......................... Off
	Path of script command ..................... /etc/pwrstatd-powerfail.sh
	Duration of command running ................ 0 sec.
	Enable shutdown system ..................... On

Action for Battery Low:

	Remaining runtime threshold ................ 300 sec.
	Battery capacity threshold ................. 35 %.
	Run script command ......................... On
	Path of command ............................ /etc/pwrstatd-lowbatt.sh
	Duration of command running ................ 0 sec.
	Enable shutdown system ..................... On

```
To configure the above setting run `sudo nano /etc/pwrstatd.conf`

I modify the following:

I want to send the shutdown signal on a specific battery percentage remaining, not automatically with a power failure. Find the following and switch to no.

```
powerfail-active = no
powerfail-shutdown = no
```

Here you can set at what percentage of battery the shutdown signal will be sent to your node. (Range is 0%-90% The default is 35%). Any devices that are not your node and are plugged in to the **battery powered plugs** will continue to be powered until the battery is totally drained. 

`lowbatt-threshold = 35`

I turn the alarm off (I'll notice the power has gone out if I am home, if not I'll get beaconcha.in notifications that my validator is offline)

`enable-alarm = off`

You can also enable email notifications. The PowerPanel user manual is on the same downloads page as the install file.


`sudo pwrstat -status` Should show an output like this. 

```
The UPS information shows as following:

	Properties:
		Model Name................... CP425SLG
		Firmware Number.............. BF01803BA41.z
		Rating Voltage............... 120 V
		Rating Power................. 255 Watt

	Current UPS status:
		State........................ Normal
		Power Supply by.............. Utility Power
		Utility Voltage.............. 116 V
		Output Voltage............... 116 V
		Battery Capacity............. 100 %
		Remaining Runtime............ 49 min.
		Load......................... 45 Watt(18 %)
		Line Interaction............. None
		Test Result.................. Unknown
		Last Power Event............. None
```

## **AC Power Recovery BIOS settings** ##

After the power is restored in order for your NUC or equilivant to *automatically power on* without having to physically press the power button you must change the following setting in the BIOS. These steps will be different depending on your machine.

The example below is for an Intel NUC:
```
To set the Intel® NUC to power on whenever a power source is connected:
Press F2 during boot to enter BIOS Setup.
Go to the Power > Secondary Power Settings menu.
Set the option for "After Power Failure" to "Power On".
Press F10 save changes and exit BIOS.
```

### **Conclusion:** ###
With the following in place a power outage will drain the UPS to 35%, at that point a shutdown signal will be send to your staking machine turning it off, modem will stay on draining the rest of the power. When power is restored the staking machine will automatically turn on, sync to the network and resume.

Trevorn.eth
