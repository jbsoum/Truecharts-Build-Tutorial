# Truecharts-Build-Tutorial
An all inclusive guide for setting up a secure apps ecosystem using Truecharts. Current guide uses TrueNAS SCALE Cobia for the build. 
Based on [these guides](https://truecharts.org/manual/SCALE/guides/getting-started) from TrueCharts, with some extra learned lessons and tips.
By the time you get to the end, you should have a completely set up, secure, and remotely accessible homelab.
You should also have everything you need to add and remove apps from this setup as needed

## Initial Requirements

1. **A system capable of running a homelab server**
    - See here for recommended system requirements from TrueNAS: [TrueNAS - SCALE Hardware Guide](https://www.truenas.com/docs/scale/23.10/gettingstarted/scalehardwareguide/)
    - See here for recommended system requirements from TrueCharts: [Truecharts - Recommended System Requirements](https://truecharts.org/manual/systemrequirements/)
    - Learn from my mistakes, and pay heed to the requirement that your **apps run on a separate SSD pool with at least 250GB**
      - I built out a bunch of apps on a regular HDD mirrored pool and could not transfer everything over to an SSD pool, using **any kind of backup / restore methodology**
    - For your boot pool, consider an SSD drive of at least 16GB
      - People used to use thumb drives for these, but I think these days, TrueNAS will [burn those out quick](https://www.truenas.com/community/threads/truenas-on-usb-drive.91273/)
    - For your data pool, consider RAIDZ1/2/3 or mirrored VDEVs
      - Internet searches will produce many opinions on whether you should go with Z1/2/3 or mirrored pools.
      - Basically, mirrored pools sacrifice capacity (at least half of your space will be redundancy) in exchange for speed
      - In terms of redundancy, RAIDZ2 is probably ideal
      - In terms of performance, mirrored vdevs are faster, and I personally use them
    - For RAM, ECC RAM has always been [highly recommended](https://www.truenas.com/community/threads/truenas-on-system-without-ecc-ram-vs-other-nas-os.107611/) for ZFS systems
      - Consider 1GB of ECC RAM per TB of total space in your pool.
      - Add an additional 16GB for the apps.
      - For example, if you have a mirrored vdev pool with 2 8TB HDD drives
        - You want 8 GB for the space, and an additional 16 GB for the apps, so at least 24GB of ECC RAM
    - Consider a system with hot swap bays
      - These make life easier, both for replacing failed drives, and installing new ones
    - Overall system minimums
      - A system with *at least* 5 HDD bays (ideally hotswap bays [seriously, trust me, hotswap bays make life much easier])
        - One for the Apps pool
        - One for the Boot pool
        - 2 for the storage pool
        - One additional as a spare slot (trust me, it's super helpful to have this!)
      - A processor with at least 4 CPU cores
      - At least 16 GB of ECC RAM, plus 1GB of ECC RAM for each TB of available space in your pool

2. **A system running TrueNAS SCALE**
    - See here for Cobia 23.10 install instructions: [TrueNAS Installation Guide](https://www.truenas.com/docs/scale/23.10/gettingstarted/)

3. **A working internet connection**
    - you should be able to ping github and truecharts *from your TrueNAS server*:
  ```
  ping https://github.com
  ```
  ```
  ping https://truecharts.org
  ```
  ```
  ping https://tccr.io
  ```

4. **A working SSH connection so that you can access your TrueNAS terminal remotely**
        -- I understand that SSH is cryptic for those unfamiliar with what it is or how it works, so I'll try to demystify the process
        -- The idea is that you, the client, wants to generate a set of keys, a *public key* and a *private key*
        -- The public key is like your address, it lets other people know who you are
        -- The private key is the secret algorithm that's used to encrypt your connection with whatever device you are connecting to, aka the server
        > **NEVER SHARE YOUR PRIVATE KEY WITH ANYONE** 🤫
        -- See below for a helpful picture from [ssh.com](https://www.ssh.com). You are the client, trying to connect to the server via SSH.
       
       ![A visual of how SSH works...you are the client, trying to ssh into the server...](https://www.ssh.com/hubfs/Imported_Blog_Media/SSH_simplified_protocol_diagram-2.png)

       Let's walk through the steps required to generate a set of host keys for your laptop, or desktop, or whatever device you want to use to connect to your server.
       You can set up more than one device, but setting that up is a bit more complex. If you're here, you're probably not ready for all of that yet.
       We're going to use OpenSSH, which is available by default on Linux or Mac. 
       If you're on Windows, here's some helpful instructions for getting OpenSSH up and running on the command line: [Microsoft - OpenSSH instructions](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui)
       There is Putty, if you want, I guess.
       Anyways, let's generate a set of public and private keys to SSH into our server.
       ```
       ssh-keygen -t ed25519 -f /path/to/save/your/private/key
       1. Generate an ssh key with ssh-keyg
   
6. **TrueCharts catalogs [added to the repository](https://truecharts.org/manual/SCALE/guides/getting-started/#adding-truecharts)**

-----

## Apps Pool Setup
### Step One: Setup your Apps Pool
    - You want this to be a pool with at least one SSD drive, with at least 250GB
    - Follow [this guide] from TrueNAS to get your pool operational

## Apps Lab Step Two: Reverse Proxy

## Apps Lab Step Three: Single Sign On

## Apps Lab Step Four: Installing other apps
