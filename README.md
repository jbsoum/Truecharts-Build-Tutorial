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
    - For your boot pool, consider an SSD drive of at least 32GB
      - People used to use thumb drives for these, but I think these days, TrueNAS will [burn those out quick](https://www.truenas.com/community/threads/truenas-on-usb-drive.91273/)
      - 16GB will be for the OS, and an additional 16GB can be used as a swap partition
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

## Step One: Set up TrueNAS SCALE Bootable USB
1. First, download the installer from here, this should be the latest Cobia build: [Download TrueNAS SCALE](https://www.truenas.com/download-truenas-scale/)
2. Next, you'll want to burn this to a thumb drive, using something like [Rufus](https://rufus.ie/) on Windows.
3. If on Windows, use [Rufus](https://rufus.ie/) to burn the iso to the USB and skip to step 8.
4. On Linux, you'll want to follow the instructions below, pulled from here: [Preparing the Install File](https://www.truenas.com/docs/scale/23.10/gettingstarted/install/installingscale/#preparing-the-install-file)
5. Connect your empty USB drive and run this command in the terminal of the Linux machine you downloaded the TrueNAS iso to:
```
lsblk -po +vendor,model
```
6. In the ```NAME``` colum, you'll see the mount path of the USB. Use that in the command below:
```
dd status=progress if=path/to/.iso of=path/to/USB
```
7. Once this command completes, you'll have a bootable usb.
8. Plug this into your server, and enter bios (usually mashing something like F12 at the bootup screen)
9. In the BIOS screen, select the option to boot from your thumb drive. This should launch the TrueNAS Installer Console Setup. 


## Step Two: Install TrueNAS from USB and set up the administrator account
See here for the guide on first time setup of TrueNAS: [TrueNAS Installer Console Setup](https://www.truenas.com/docs/scale/gettingstarted/install/installingscale/#using-the-truenas-installer-console-setup)
1. Select **Install/Upgrade**

![SCALE Install Main Screen](https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallMainScreen.png)

2. Select the desired install drive

![SCALEInstallDriveScreen](https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallDriveScreen.png)
    
- You want to make sure you select the SSD drive you want to use as your boot drive

Select **Yes**.

![SCALEInstallWarningScreen](https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallWarningScreen.png)

3. Select **Fresh Install** to do a clean install of the downloaded version of TrueNAS SCALE. This erases the contents of the selected drive!

![SCALEInstallUpgradeFresh](https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallUpgradeFresh.png)

When the operating system device has enough additional space, you can choose to allocate some space for a swap partition to improve performance. Select **Create Swap** and press Enter.

![SCALEInstallPartitionScreen](https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallPartitionScreen.png)

When existing versions of TrueNAS are present on the device, you can choose **Install in new boot environment** to create a partition or **Format the boot device** to remove previous boot environments.

![SCALEInstallUpdateMethodSelection](https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallUpdateMethodSelection.png)

4. Select option **1 Administrative user (admin)** and then **OK** to install SCALE, and create the admin user account. SCALE Bluefin has implemented rootless login. Create an admin account and password. The system retains root as a fallback but it is no longer the default. This account has full control over TrueNAS and is used to log in to the web interface. Set a strong password and protect it. We do not recommend selecting **3 Configure using Web UI**.
![SCALEInstallerConsoleSetupAdminAccount](https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallerConsoleSetupAdminAccount.png)

Next, enter a password for the new admin user.

![SCALEInstallerConsoleSetupAdminPassword](https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallerConsoleSetupAdminPassword.png)

5. Select Boot via UEFI at the TrueNAS Boot Mode prompt, then select OK and press Enter to begin the installation.

After following the steps to install, reboot the system and remove the install media.

-----

## Step 3: Accessing the Web GUI
1. After removing the USB and rebooting the system, you should be presented with the Console Setup screen

![ConsoleSetupMenuSCALE](https://www.truenas.com/docs/images/SCALE/CLI/ConsoleSetupMenuSCALE.png)

2. At the top of this screen is the local IP address of the Web GUI

3. From any computer on the same network as your TrueNAS server, you should be able to enter that address and see the Web GUI Login Screen
    - For example, if your Web GUI is at ```https://192.168.1.10:443```, enter that into a web browser on a computer connected to the same network as your server
  
![LoginScreenSCALE](https://www.truenas.com/docs/images/SCALE/Login/LoginScreenSCALE.png)

4. Enter the administrator username and password you set up in **Step Two**

5. At this point, you should be logged into the TrueNAS Web GUI!

6. Let's take a quick break, we've accomplished a lot at this point 😌👍🏼

7. *After your break*, check this guide out for a detailed explaination of everything you can do in the Web GUI: [TrueNAS SCALE - UI Reference Guide](https://www.truenas.com/docs/scale/scaleuireference/) 

-----

## Step One: Set up the SCALE Network

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
