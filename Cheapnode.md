# Run a Rocketpool node on the cheap by going used. ~$330USD #
With LEB8's around the corner I thought I'd write up a cheap solution for home staking hardware. I’ll go over price and performance numbers. Great for those on a budget, or for people looking for a testnet/fallback machine.


First I picked up this used MiniPC from Ebay:

![HPelitedesk800mini](/../photos/800-g32.jpg)

### **HP EliteDesk 800 G3 Mini** ### 
### [Review with photos](https://www.servethehome.com/hp-elitedesk-800-g3-mini-ce-review-project-tinyminimicro/) ###

System Specs:
```
Released Date: 2017
Processor: 6th gen Intel i5-6500T 2.50GHz 
Cores: 4
Threads: 4
TDP: 35W 
Ram: 8GB DDR4 2400 SO-DIMM
PCIe 3.0 M.2 slot
Power adapter included
```

The seller had 50 of these units for sale. No SSD was included which makes me believe these were all from an office environment where the drives were removed/destroyed. Carefully read any used listing to confirm that a CPU and Power adapter are included. 

### **Elitedesk Price: $69.99 + $22.96 shipping = $89.95USD** ###


## **Memory** ##
With 4 cores and 4 threads this machine will be very capable to run your rocketpool node, a TDP of 35W means it will use a small amount of power while staking. The machine shipped with a single stick of 8GB DDR4 2400 SO-DIMM. This is below the recommended amount of ram in the rocketpool docs. At the minimum an additional 8GB stick was required. I didn’t want to be [limited in what execution client and consensus client I could run](https://docs.rocketpool.net/guides/node/local/hardware.html) so I purchased 32GB of RAM. This is also the maximum supported by the HP Elitedesk 800 G3 Mini.

**CORSAIR Vengeance 32GB (2 x 16GB) 260-Pin DDR4 SO-DIMM DDR4 2400 (PC4 19200) Memory**

### **Memory Price: $79.99USD** ###


## **M.2 Nvme** ##

Due to the data transfer demands of running a rocketpool node I went with a suitable 2TB NVMe drive that will meet the demands of staking from [Yorrick’s Overview](https://gist.github.com/yorickdowne/f3a3e79a573bf35767cd002cc977b038) of various options. The Elitedesk 800 supports PCIe 3.0x so I went with:


**Samsung 970 evo plus 2TB PCIe Gen 3.0 x4 NVMe 1.3**
### **NVMe Price: $159.99USD** ###







![RAM and NVMe installed](/../photos/IMG_3583.png)<sub> The fan unit flips up easily to install the Memory. SSD cradle was removed and there is clearance for a NVMe Headsink. </sub>


## **Performance** ##
I installed Debian 11 (bullseye) and Rocketpool. After ETH1/ETH2 Sync completed I restored my wallet.
For a total of approximately $330.00USD This is the performance you can expect: 

![Grafana Dashboard](/../photos/elitedesk800.png)


Running Lighthouse/Nethermind:
```
IOPS Read = 109K (recommended min. 15k for Rocketpool)
IOPS Write = 36.4K (recommended min. 5k for Rocketpool)
CPU load averaging under 40%
Measured Avg Power over 24hr period: 13.05W 
RAM usage is 22GB (note: running nethermind)
CPU Temperature under 40C
NVMe Temperature mid 40’s
SSD Latency under 6ms
I/O Wait time under 45ms 
```

Add NVMe Temp to your gradana dashboard following this [guide by Patches](https://gist.github.com/jshufro/65160a680076224d0294d1d6f1a0fa97)




*Other considerations: Used miniPCs like this one may have been run 24/7 for years so I blew it out with compressed air, removed the heatsink, applied new thermal paste to the CPU. Updating the BIOS can be done from in the BIOS with a network connection. During initial sync CPU load was pegged at 100% and the NVMe temps spiked to the mid 70’s C so I also added a cheap NVMe heatsink.*

Trevorn.eth
