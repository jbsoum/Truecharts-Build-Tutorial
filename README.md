# Truecharts-Build-Tutorial
An all inclusive guide for setting up a secure apps ecosystem using Truecharts. Current guide uses TrueNAS SCALE Cobia for the build. 

## Initial Requirements
1. A system running TrueNAS SCALE
  - See here for Cobia 23.10 install instructions: [TrueNAS Installation Guide](https://www.truenas.com/docs/scale/23.10/gettingstarted/)
  - See here for recommended system requirements: [Truecharts: Recommended System Requirements](https://truecharts.org/manual/systemrequirements/)
  - Learn from my mistakes, and pay heed to the requirement that your apps run on a separate SSD pool with at least 250GB
    - I built out a bunch of apps on a regular HDD mirrored pool and could not transfer everything over to an SSD pool, using **any kind of backup / restore methodology**
    - So I had to rebuild everything from scratch, using a fresh SSD pool.
    - Figured I would make a guide this time around.
  - For your TrueNAS system, if you are sharing data with your apps, you want at least one pool of data, likely on mechanical drives using RAIDZ1/2/3 or mirrored VDEVs
    - Internet searches will produce many opinions on whether you should go with Z1/2/3 or mirrored pools.
    - Basically, mirrored pools sacrifice capacity (at least half of your space will be redundancy) in exchange for speed
    - In terms of redundancy, RAIDZ2 is probably ideal
    - In terms of performance, mirrored vdevs are faster, and I personally use them
  - For your boot pool, consider an SSD drive of at least 50GB
    - People used to use thumb drives for these, but I think these days, TrueNAS will [burn those out quick](https://www.truenas.com/community/threads/truenas-on-usb-drive.91273/)
  - Consider a system with hot swap bays
    - These make life easier, both for replacing failed drives, and installing new ones
      - One slot for your Apps pool
      - One slot for your Boot pool
      - Remaining slots for your data pool (most likely mechanical drives)
      - Save one or two slots for hot swap bays, to change failed drives if you need to
     
## Apps Lab Step One: Data Sharing

## Apps Lab Step Two: Reverse Proxy

## Apps Lab Step Three: Single Sign On

## Apps Lab Step Four: Installing other apps
