# Protecting your Rocketpool node from power outages on the cheap #

An uninterruptible power supply (UPS) is a device that allows a computer to keep running for at least a short time when incoming power is interrupted. Along with the obvious power connection, a secondary connection via USB can trigger a **graceful shutdown** to your staking machine in the event of a power loss. Without this in place, a **hard shutdown** can cause corruption of program and operating system files. With Ice storms, hurricanes, and terrible power utility infrastructure I wanted to protect against these events the cheapest way possible. My goal wasn’t to stay up with battery power for any meaningful length of time. I just wanted to gracefully shut down my staking machine, then to automatically start up and continue normal operation when power was restored. Total hands-off, worry free operation. 

Most staking machines use under 25W of power so the load requirements are pretty low. I looked for the cheapest UPS **with USB connectivity** needed for a graceful shutdown command to be sent to a PC. There are less expensive UPS options but they lack this function.

## Cyberpower CP425SLG ## 
### MSRP: $57.95USD ###
![CP425SLG](https://github.com/trevhub/rocketpool/blob/b245bf1c5f5521c3991cfa17dadb3f8456bc3d8d/CP425SLG_F2.jpg)

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

Total Load = 38W
```
38W works out to 15% load on the CP425SLG and will give me 57 minutes of uptime. ([or 4% Load, 6hr up time on the $759 APC Smart-UPS 1500VA](https://www.apc.com/us/en/product/SMT1500C/apc-smartups-line-interactive-1500va-tower-120v-8x-nema-515r-outlets-smartconnect-port+smartslot-avr-lcd/))

The connections:
![ups connections](https://github.com/trevhub/rocketpool/blob/b245bf1c5f5521c3991cfa17dadb3f8456bc3d8d/UPSconnections.png)
The side of the CP425SLG labeled **SURGE** is for *surge protection only* and doesnt run on battery power during an outage. I have my TV, Stereo, Laptop plugged in to those outlets. 

## **Installing the Cyberpower software on your node:** ##

My options were **[Cyberpowers PowerPanel® Personal](https://www.cyberpowersystems.com/product/software/power-panel-personal/powerpanel-for-linux/)** to configure and monitor my UPS or **[Network UPS Tools](https://networkupstools.org/)**. NUT is a free and open source UPS program licensed under the terms of GNU Public License (GPL). NUT works with 170 different manufacturers, and several thousands of models. 

### *Security Note: You are about to install software on your staking machine, always proceed with caution and verify URLs.* ###

I went with Cyberpowers Powerpanel.
https://www.cyberpowersystems.com/product/software/power-panel-personal/powerpanel-for-linux/

choose your install file.
![powerpanel](https://github.com/trevhub/rocketpool/blob/b245bf1c5f5521c3991cfa17dadb3f8456bc3d8d/powerpanel.png)

Log into your staking machine and download the `.deb` file replacing `[CORRECT URL]` with the address

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

`enable-alarm = no`

Additionally, the amount of time taken for the UPS itself to shutdown (along with everything else powered by it) after a Battery Low Event can be modified (In seconds)

`shutdown-sustain = 300`

You can also enable email notifications. The PowerPanel user manual is on the same downloads page as the install file.

To make the changes take effect:
`sudo /etc/init.d/pwrstatd restart`

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
		Remaining Runtime............ 57 min.
		Load......................... 38 Watt(15 %)
		Line Interaction............. None
		Test Result.................. Unknown
		Last Power Event............. None
```

## **AC Power Recovery BIOS settings** ##

After the power is restored in order for your NUC or equivilant to *automatically power on* without having to physically press the power button you must change the following setting in the BIOS. This requires a monitor and keyboard connected during the boot sequence. These steps will be different depending on your machine. Common keys to enter the BIOS are DEL, F10, F12 and F1

The example below is for an Intel NUC:
```
To set the Intel® NUC to power on whenever a power source is connected:
Press F2 during boot to enter BIOS Setup.
Go to the Power > Secondary Power Settings menu.
Set the option for "After Power Failure" to "Power On".
Press F10 save changes and exit BIOS.
```
Note: a full power down of your UPS (the timing of which is controlled by the above mentioned variable `shutdown-sustain`) *may* be required for your staking hardware to actually automatically power on when power is restored to your UPS. If you do not require your UPS to shutdown after a certain amount of time, `turn-ups-off` can be changed to `no`. *However*, this is not recommended and should be tested to ensure that your staking hardware actually powers back up once power is restored to the UPS.

### **Conclusion:** ###
With the following in place, a power outage will drain the UPS to 35%. At that point, a shutdown signal will be sent to your staking machine, turning it off. Any other devices plugged into the battery-powered outlets, such as a modem, will stay on for 300 additional seconds, after which the entire UPS will turn off. When power is restored, the staking machine will automatically turn on, sync to the network, and resume operation.

Trevorn.eth
