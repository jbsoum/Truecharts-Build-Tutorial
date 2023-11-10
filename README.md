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

5. **TrueCharts catalogs [added to the repository](https://truecharts.org/manual/SCALE/guides/getting-started/#adding-truecharts)**


-----

## Step One: Set up the Scale Network



1. **Set up the TrueNAS Scale Network**
    - Go to *Network*, and for the interface (ethernet port) which is connected to the internet, all the way to the right, click the little pencil icon to Configure
    - Under *Aliases*, set the IP Address to your desired local IP address for your server
        - This IP can or cannot be outside of your DHCP range
        - The DHCP range on your router is the range of IP addresses reserved for automatic assignment when someone new connects to the network
        - If you're going to run apps off of this server, you really should have a static IP
        - We'll set it here, in TrueNAS, then set it up on your router as well
    - Save these settings
    - Still in the *Network* tab of the Web GUI, click on *Settings* to the right of *Global Configuration*
    - The *Hostname* and *Domain* define the local address of your server
        - For example, if you select ```truenas``` for *Hostname* and ```local``` for *Domain*, you can access your server at ```http://truenas.local```
    - For *Nameserver 1* and *Nameserver 2*, you can use ```1.1.1.1``` and ```1.0.0.1```, these are Cloudfare's default DNS servers
        - When you get DNS services from Cloudfare, they give you special DNS domains for your server, you could use those too I guess, but I don't think it matters.
    - For *IPv4 Default Gateway*, enter the IP address you would use to log into your router. That's your default gateway. For many, that's ```192.168.1.1```.
    - For *Outbound Network*, make sure *Allow All* is selected.
    - Save these settings.
2. **Set a static IP address for your server on your router**
    - This is the IP address you set for the interface above in Step 2
    - You typically also need the MAC address for your interface
    - You can get that running the command below in the TrueNAS shell under *System Settings*
    - You should a bunch of sections, one of them will be labeled with the name of the interface you set an IP address for in Step 2 above.
    - In that section, there should be a line that looks something like ```ether ax:3g:8c:04:07:v8```. That string of alphanumeric characters and colons is your MAC address
    - You may also need your hostname, which you also set in Step 2 above.
```
ifconfig -a
```

-----

## Step Two: Set up the root account and Terminal Access
- I understand that SSH is cryptic for those unfamiliar with what it is or how it works, so I'll try to demystify the process
- The idea is that you, the client, wants to generate a set of keys, a *public key* and a *private key*
- The public key is like your address, it lets other people know who you are
- The private key is the secret algorithm that's used to encrypt your connection with whatever device you are connecting to, aka the server
> **NEVER SHARE YOUR PRIVATE KEY WITH ANYONE** 🤫
- See below for a helpful picture from [ssh.com](https://www.ssh.com). You are the client, trying to connect to the server via SSH.
![A visual of how SSH works...you are the client, trying to ssh into the server...](https://www.ssh.com/hubfs/Imported_Blog_Media/SSH_simplified_protocol_diagram-2.png)

Let's walk through the steps required to generate a set of host keys for your laptop, or desktop, or whatever device you want to use to connect to your server.
You can set up more than one device, but setting that up is a bit more complex. If you're here, you're probably not ready for all of that yet.
We're going to use OpenSSH, which is available by default on Linux or Mac. 

If you're on Windows, here's some helpful instructions for getting OpenSSH up and running on the command line: [Microsoft - OpenSSH instructions](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui)
There is Putty, if you want, I guess.

1. **Forward a port for your SSH connection on your router**
    - You'll want to forward *some port above 40000*,
    - Use TCP only,
    - For IP address to forward to, use the static IP address you selected for your server.
    - We'll come back to this port forwarding screen for your router to give other apps remote access as needed
  

2. **Set up the SSH Service in TrueNAS**
   - Go To *System Settings* -> *Services*
   - Make sure the *SSH*, *NFS*, and *SMB* Services are *Running*, and are set to *Start Automatically*
   - On the *SSH* line, all the way to the right, click the little pencil icon to Configure SSH
   - Set up the TCP port for SSH to be *something else besides 22*.
       - ***Why are we doing this?*** Well, 22 is the default port for SSH connections, and it's a popular one for hackers to try and break into
       - We are going to keep that port closed, and open up another random port instead for SSH connections
       - This is more secure
    - Enable *Allow TCP Port Forwarding*
    - Save these settings

3. **Forward the port you selected in Step 1 on your router**
    - This step is different depending on your router  
4. **Generate an ssh key with ssh-keygen**
```
ssh-keygen -t ed25519 -c <your@email.address> -f /path/to/save/your/private/key
```
- **-t ed25519**: The encryption algorithm - this is the most advanced one available in OpenSSH
- **-c <your@email.address>**: Key comment - this will be appended to the end of your public key
    - it's for your records, it can be whatever you want, or you can omit it
- **-f /path/to/save/your/private/key**: The Private Key File Name - **This is the thing you don't want to share with anyone!**
    - The typical place to save these is in the ```.ssh``` directory.
    - This can be found in ```/home/<YourUsername>/.ssh/``` on Linux and MacOS
    - On Windows, this can be found in ```C:\Users\<YourUsername>\.ssh\```
    - I recommend naming the key something easy for you to remember. This helps to keep track of multiple keys.
        - For example, ```/home/<YourUsername>/.ssh/root@example.com```
        - This tells us we use this key to login to our server at ```example.com``` as the user ```root```.
    - OpenSSH will create an additonal file called **/path/to/save/your/private/key _.pub_**.
        - For example, if you named your private key  ```/home/<YourUsername>/.ssh/root@example.com```,
        - then your public key will be ```/home/<YourUsername>/.ssh/root@example.com.pub```

-----

## Step Two: Set up your pools
If you've been following along so far, you should be able to go to Storage in the TrueNAS GUI and see *at least* 3 unassigned disks.
One of those disks should be an SSD that's at least 256GB, and the other 2+ should be mechanical drives. 

### SSD Pool - where your apps will go
Click the **Create Pool** button in the ***Storage*** tab of the TrueNAS GUI.



## Apps Pool Setup
### Step One: Setup your Apps Pool
    - You want this to be a pool with at least one SSD drive, with at least 250GB
    - Follow [this guide] from TrueNAS to get your pool operational

## Apps Lab Step Two: Reverse Proxy

## Apps Lab Step Three: Single Sign On

## Apps Lab Step Four: Installing other apps
