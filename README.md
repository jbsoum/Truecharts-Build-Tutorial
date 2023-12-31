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

## Part I: Setting up the TrueNAS System

-----

### Step One: Set up TrueNAS SCALE Bootable USB
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


### Step Two: Install TrueNAS from USB and set up the administrator account
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

### Step 3: Accessing the Web GUI
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

### Step Four: Set up the SCALE Network

1. **Set up the TrueNAS Scale Network**
    - Go to *Network*, and for the interface (ethernet port) which is connected to the internet, all the way to the right, click the little pencil icon to Configure

![NetworkInterfacesWidget](https://www.truenas.com/docs/images/SCALE/Network/NetworkInterfacesWidget.png)

- Under *Aliases*, set the IP Address to your desired local IP address for your server
    - This IP can or cannot be outside of your DHCP range
    - The DHCP range on your router is the range of IP addresses reserved for automatic assignment when someone new connects to the network
    - If you're going to run apps off of this server, you really should have a static IP
    - We'll set it here, in TrueNAS, then set it up on your router as well
- Save these settings

- Still in the *Network* tab of the Web GUI, click on *Settings* to the right of *Global Configuration*

![GlobalConfiguration](https://www.truenas.com/docs/images/SCALE/Network/GlobalConfiguration.png)

- The *Hostname* and *Domain* define the local address of your server
    - The *Hostname* is the network name of your server
        - ```truenas``` is a typical choice  
    - The *Domain* is the network address of your server. You can enter additional domains as well
        - I like to start with ```truenas.local``` for the domain, and then if I have an external domain I would access remotely, I would add that under *Additional Domains*.  
    - For example, if you select ```truenas``` for *Hostname* and ```local``` for *Domain*, you can access your server at ```http://truenas.local```
- For *Nameserver 1* and *Nameserver 2*, you can use ```1.1.1.1``` and ```1.0.0.1```, these are Cloudfare's default DNS servers
    - When you get DNS services from Cloudfare, they give you special DNS domains for your server, you could use those too I guess, but I don't think it matters.
- For *IPv4 Default Gateway*, enter the IP address you would use to log into your router. That's your default gateway. For many, that's ```192.168.1.1```.
- For *Outbound Network*, make sure *Allow All* is selected.
- Save these settings.

![EditGlobalConfiguration](https://www.truenas.com/docs/images/SCALE/Network/EditGlobalConfiguration.png)

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

### Step Five: Set up Terminal Access
- The idea here is that you're unlikely to have a monitor attached to your server at all times
- There is a Shell you can access via the web GUI, but you should not expose that to the internet, and it's a bit finicky
- So, without a monitor, how can you interact with the terminal? What if you're away from your desk?
- Enter SSH! You don't want to open your server up to the internet, but *if we can create a secure tunnel between the server and whatever machine we're on, that should be pretty secure*!
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
If you can't stand OpenSSH, there is also Putty, if you want, I guess.

1. **Forward a port for your SSH connection on your router**
    - This step is different for everyone depending on your router, search for ```Port forwarding <YourRouterNameAndModel>```
    - You'll want to forward *some port above 40000*,
       - ***Why are we doing this?*** Well, 22 is the default port for SSH connections, and it's a popular one for hackers to try and break into
       - We are going to keep that port closed, and open up another random port instead for SSH connections
       - This is more secure
    - Use TCP only,
    - For IP address to forward to, use the static IP address you selected for your server.
    - We'll come back to this port forwarding screen for your router to give other apps remote access as needed
  

3. **Set up the SSH Service in TrueNAS**
   - Go To *System Settings* -> *Services*

![ServicesSCALE](https://www.truenas.com/docs/images/SCALE/SystemSettings/ServicesSCALE.png)

   - Make sure the *SSH*, *NFS*, and *SMB* Services are *Running*, and are set to *Start Automatically*
   - On the *SSH* line, all the way to the right, click the little pencil icon to Configure SSH.
   - Set the TCP port for SSH to be *the port you forwarded on your router in Step 1*.
    - Enable *Allow TCP Port Forwarding*
    - Do NOT enable *Password Authentication*
        - **Why are we doing this?**
        - We are going to set up access via SSH, so we can keep our system extra secure by making sure the *only way* to access the system remotely is with a valid SSH key registered with the server
    - Save these settings

![ServicesSSHBasicSettingsGenOptionsSCALE](https://www.truenas.com/docs/images/SCALE/SystemSettings/ServicesSSHBasicSettingsGenOptionsSCALE.png)

4. **_On the client machine (the machine you'll be accessing the server from)_, generate an ssh key with ssh-keygen**
```
ssh-keygen -t ed25519 -c <your@email.address> -f /path/to/save/your/private/key
```
- **-t ed25519**: The encryption algorithm - this is the most advanced one available in OpenSSH
- **-c \<your@email.address\>**: Key comment - this will be appended to the end of your public key
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
- After the key is generated, you'll see a cool ASCII graphic, then you'll be asked:

> Enter passphrase (empty for no passphrase): [Type a passphrase]

> Enter same passphrase again: [Type passphrase again]

- If you leave both of these fields empty, then your SSH key can be used without a password. **I DO NOT RECOMMEND THIS**.
- Lock your key with a secure password. That way, no one can use your key to gain access to your server without that passphrase.

5. **Register your key with SSH Agent**
- Run the below command to get ssh-agent running:
```
eval "$(ssh-agent -s)"
```

- You should see something like ```Agent pid 12345``` returned.
- Next, add the key to ssh-agent with the following command:
```
ssh-add /path/to/save/your/private/key
```
> You want to register your **private key** here, not the public one ending in *.pub*

     
6. **Still on the client machine, run the below command, which will return your client machine's public key**
```
cat /path/to/save/your/private/key.pub
```
- So, whatever your public key's name is, we are going to return its contents to the terminal screen
> Do NOT do this with your private key! 🤫
- Copy this by selecting it in the terminal, holding ```CTRL``` + ```SHIFT```, and press ```C```.
     
7. **Register this key with your admin account**
    - Note that if you are upgrading from a previous version of FreeNAS / TrueNAS, you're probably using hte ```root``` account as your admin account
    - This is being deprecated in future releases, so be prepared to use a separate admin account for all of your root stuff as this guide suggests eventually...
    - Go to  *Credentials* -> *Local Users*, and select your admin account for editing

![UserScreenUserDetails](https://www.truenas.com/docs/images/SCALE/Credentials/UserScreenUserDetails.png)

- In this screen, we are going to paste the public key we copied in Step 5, and paste it in the *Authorized Keys* section

![AddUserHomeDirAuthSCALE](https://www.truenas.com/docs/images/SCALE/Credentials/AddUserHomeDirAuthSCALE.png)

- Make sure *SSH Password Login* is **UNCHECKED**
    - **Why are we doing this**?
    - Wellll, we don't want to use passwords to gain SSH access with the server.
    - The only way in should be a secure key known by the server, further secured with a password on the client side.

8. **Now, you should be able to log into your server remotely using the command below**:
```
ssh <YourAdminAccount>@<YourServerDomain> -p <YourSSHPort>
```

-----

## Part II: Pool Setup
If you've been following along so far, you should be able to go to Storage in the TrueNAS GUI and see *at least* 3 unassigned disks.
One of those disks should be an SSD that's at least 256GB, and the other 2+ should be mechanical drives. 

### SSD Pool - where your apps will go
Click the **Create Pool** button in the ***Storage*** tab of the TrueNAS GUI.

1. **Select a name for your SSD pool**
    - People tend to choose something simple, all lower case without dashes or underscores
    - For the purpose of this tutorial, well assume this pool is called ```ssd```
    - Leave the drive unencrypted
    - Click *Next*

![PoolCreationWizardGeneralInfo](https://www.truenas.com/docs/images/SCALE/Storage/PoolCreationWizardGeneralInfo.png)

2. **Add the SSD to your pool**
    - For *Layout*, select *Stripe*
    - Click *Manual Disk Selection* under *Advanced Options* 
    - In the top right corner, click *Add* to add the only VDEV we'll need for this pool
    - Select the SSD disk for your pool on the left, and drag it into your new VDEV on the right
    - If you have a lot of disks, feel free to use the disk filters to help parse down the list to find your disk
        - We will be setting up backups of our app directory, so disk redundancy isn't super necessary for this pool
        - If you must, you can set up a 2-way or 3-way mirrored VDEV
    - Click *Save Selection*, then click *Save and Go To Review*
     
![ManualSelectionScreen](https://www.truenas.com/docs/images/SCALE/Storage/ManualSelectionScreen.png)

3. **Review your selection and create your pool**
    - *Pool Name* should be what we set above, (eg ```ssd``` as in our example)
    - *Data* under *Topology* should say something like ```1 x STRIPE|1 x #.## GiB (SSD)```
    - If everything looks good, go ahead and click *Create Pool*

![PoolCreationWizardReviewScreen](https://www.truenas.com/docs/images/SCALE/Storage/PoolCreationWizardReviewScreen.png)

### Storage Pool - where your data will go
At this point, we assume you have at least 2 unassigned disks left, all mechanical drives.
For your storage pool, consider RAIDZ1/2/3 or mirrored VDEVs
  - Internet searches will produce many opinions on whether you should go with Z1/2/3 or mirrored pools.
  - Basically, mirrored pools sacrifice capacity (at least half of your space will be redundancy) in exchange for speed
  - In terms of redundancy, RAIDZ2 is probably ideal
  - In terms of performance, mirrored vdevs are faster, and I personally use them


1. **Select a name for your storage pool**
    - People tend to choose something simple, all lower case without dashes or underscores
    - For the purpose of this tutorial, well assume this pool is called ```storage```
    - Leave the drive unencrypted
    - Click *Next*
  
2. **Add the HDDs to your storagepool**
    - For *Layout*, select your chosen redundancy framework (hopefully NOT Stripe if you've been listening to me!)
    - Under *Automated Disk Selection*, select the *Disk Size* that matches the size of your mechanical drives
    - Under *Width*, this depends on your redundancy framework
        - RAID Z1/2/3 is essentially 1 VDEV per pool
        - Mirrored will create one VDEV per mirror
        - Mirrors need to have at least two disks per VDEV
        - So for a 2 way mirror (50% of HDD capacity spent on redundancy), select *2* for *Width*
        - For a 3 way mirror (66.7% of HDD capacity spent on redundancy), select *3* for *Width*
    - The *Number of VDEVs* will be limited by the number of disks you have and your redundancy framework
    - Click *Save and Go To Review*
  
![PoolCreationWizardDataScreen](https://www.truenas.com/docs/images/SCALE/Storage/PoolCreationWizardDataScreen.png)

3. **Review your selection and create your pool**
    - *Pool Name* should be what we set above, (eg ```storage``` as in our example)
    - *Data* under *Topology* should say something like ```<V> x STRIPE| <W> x #.## GiB (SSD)```
        - <V> represents the number of VDEVs in your pool
        - <W> represents the width you selected
    - If everything looks good, go ahead and click *Create Pool*
  
-----

## Part III: Set up the Apps Cluster

-----

### Step One: Set up your Apps Pool

1. In the Web GUI, go to the Apps tab

![AppsInstalledAppsScreenNoApps](https://www.truenas.com/docs/images/SCALE/Apps/AppsInstalledAppsScreenNoApps.png)

2. The first time you open the Applications screen, it displays an *Apps Service Not Configured* status on the screen header.

![AppsServiceNotConfigured](https://www.truenas.com/docs/images/SCALE/Apps/AppsServiceNotConfigured.png)

3. Click *Settings* -> *Choose Pool* and select the ```ssd``` pool we created above for our Apps cluster

![AppsChoosePoolForApps](https://www.truenas.com/docs/images/SCALE/Apps/AppsChoosePoolForApps.png)

4. After an Apps storage pool is configured, the status changes to *Apps Service Running*.


### Step Two: Add the TrueCharts catalogs to your system
See [this guide](https://truecharts.org/manual/SCALE/guides/getting-started/#adding-truecharts) from TrueCharts to get the right catalogs added
- You'll need at least ```stable```, ```enterprise```, and  ```operators``` trains
- The ```incubator``` train is for what can be considered 'beta' charts, use them at your own peril.

### Step Three: Add the base TrueCharts Operator charts
- In the TrueNAS Web GUI, go to the *Apps* tab, then click on *Discover Apps*
- You're going to install 3 charts from this screen: ```cloudnative-pg```, ```prometheus-operator```, and ```cert-manager```
- You'll find these apps on the TrueCharts operators train
    - Search for the chart name in the *Search Bar*
    - Click on the tile for the app that shows up
    - Click *Install*
    - The Chart Config screen will show up, click *Install* and leave the settings as default for these charts

You should see something like below in the Apps screen when you're done (3 apps that show *Running*)

![An image of the 3 operator apps running](link.img)

### Step Four: Install Heavyscript on your sever

Remember those fancy ssh credentials we made in a prior step? It's time to use those!

Open up a terminal or command line on the machine you set up security keys on (hopefully this one, lol)

ssh into your server using the following command:

```
ssh <admin>@your-server.domain -p <YourSSHPort>
```
- \<admin\>@your-server.domain: \<admin\> is your admin account you set up when you installed TrueNAS (or root if you upgraded from an older version)
- \<YourSSHPort\>: this is the port you set up on your TrueNAS SSH service, and forwarded on your router for this purpose
- If everything works, it should ask you for your SSH passphrase. Enter it, and you're logged into your TrueNAS terminal!

If that DIDN'T work, try the same command, using your server Static IP address we set above in TrueNAS's network configuration, and also on your router:

```
ssh <admin>@Your.Server.IP.Address -p <YourSSHPort>
```

If that ALSO DIDN'T work, something's wrong!
- If the IP address didn't work, is the machine you're trying to connect from on the same network as your server?
- Are you ***sure*** you forwarded that port on your router?
- Are you ***sure*** that's the same port you set up in *System Settings* -> *Services* -> *SSH*?
- Are you ***sure*** that's your TrueNAS Static IP address? Set up correctly in TrueNAS *Network* tab AND on your router? (check ```ifconfig -a``` in *System* -> *Shell*)
- Are you ***sure*** you entered your SSH passphrase correctly?
- Are you ***sure*** you copied your SSH public key's contents *entirely* and pasted them into the *Authorized Keys* section of your admin account in *Credentials* -> *Local Users*?

If that DID work, you should be greeted with a welcome message from your server, and a command prompt that looks something like this:
```
<admin>@<Hostname>[~]:
```
- \<admin\>: this is your administrator account (or root)
- \<Hostname\>: this is the hostname you set up in your *Network* settings
- [~]: this means you are at the ```home``` directory of your administrator account on your server. Whatever path you are currently at will be displayed in these brackets

NOW it's time to install [HeavyScript](https://github.com/Heavybullets8/heavy_script).
What is HeavyScript? It's a bash script for managing TrueNAS charts, very helpful companion with TrueCharts

From their github, you can install HeavyScript from the command line with just one line:
```
curl -s https://raw.githubusercontent.com/Heavybullets8/heavy_script/main/functions/deploy.sh | bash && source "$HOME/.bashrc" 2>/dev/null && source "$HOME/.zshrc" 2>/dev/null
```
Copy and paste that into your logged in ssh terminal, and press ```Enter```.

From [their github](https://github.com/Heavybullets8/heavy_script#how-to-install), this command will:

- Download HeavyScript, then place you on the latest release
- Place HeavyScript in ```/root```
- Make HeavyScript executable
- Allow you to run HeavyScript from any directory with ```heavyscript```

Check to see if the install went successfully by typing ```heavyscript``` into your SSH terminal. If you see HeavyScript launch, you're good to continue!

Let's finally add a Cron Job for HeavyScript. We want it to run automatically once a day and:

- update the HeavyScript app
- check for updates
- Create and store a backup of the apps cluster

Let's go to *System* -> *Advanced Settings*:

![SystemAdvancedScreen](https://www.truenas.com/docs/images/SCALE/SystemSettings/SystemAdvancedScreen.png)

In the *Cron Jobs* widget, click *Add*:

![AdvancedSettingsCronJobWidget](https://www.truenas.com/docs/images/SCALE/SystemSettings/AdvancedSettingsCronJobWidget.png)

In the *Add Cron Job* screen:

- *Run as User* = your administrator account
- *Schedule* = run daily at some time
- *Hide Standard Output*= unchecked (for email notifications)
- *Hide Standard Error*= unchecked (also for email notifications)
- *Command* = Use this to backup, update HeavyScript, and update any apps

```
bash /root/heavy_script/heavy_script.sh update --backup 14 --concurrent 10 --prune --rollback --sync --self-update
```

![HeavyScript Cron Job Example](https://user-images.githubusercontent.com/20793231/229404447-6836ff1f-ba28-439e-99fe-745371f0f24c.png)

> Confused about Cron Jobs? Wtf are those numbers, anyway? See below for a graphic.

![crontab-syntax](https://www.hostinger.com/tutorials/wp-content/uploads/sites/2/2021/09/crontab-syntax.webp)


-----

## Part IV: Set up a domain name for your server
Okay, great job! You made it really far!

Let's take a break from server stuff and create a domain name for our server!

What's that? You don't know anything about domain names?

### Step 1: Getting a domain name

Take a look at this quick lesson from [Free Code Camp](https://www.freecodecamp.org): [What is DNS?](https://www.freecodecamp.org/news/what-is-dns/)

Basically, DNS is the system the internet uses to map IP addresses to domain names like https://your-cool-server.xyz

First, you're going to need a domain name. If you don't want to pay for one, you can use free services like [Freenom](https://www.freenom.com).
It should be noted that you don't have ownership or transfer rights over the domain you get from Freenom.

If you want your own snazzy domain, check out [Namecheap](https://www.namecheap.com/)

Once you have a domain, we're going to manage DNS services through [Cloudfare](https://www.cloudflare.com/). Don't worry, it's free for what we're using it for!

Here are some instructions from Namecheap on [how to use a 3rd party DNS service with your Namecheap domain](https://www.namecheap.com/support/knowledgebase/article.aspx/767/10/how-to-change-dns-for-a-domain/).
The process for other domain providers is similar.

You can also transfer ownership rights from Namecheap to Cloudfare, if you'd like. Here's a [guide from Cloudfare](https://blog.cloudflare.com/a-step-by-step-guide-to-transferring-domains-to-cloudflare/) on how to do that.

### Step 2: Resolving the domain through [Cloudfare](https://dash.cloudflare.com) DNS

Yeah, we're using Cloudfare. Why? Well, it's the only DNS service Truecharts currently supports, plus it's free for what we need it for. 
As TrueCharts grows their ecosystem, I'm sure there will be other options. 

Basically, we're going to add an ```A Record``` to link your server's IP address to your newly acquired domain

Later, we're going to add a ```CNAME Record``` for each app, with the URL convention \<app\>.\<your-snazzy-domain\>.com
The CNAME Record forwards requests made to a particular alias to a hostname or IP address of our choosing.

1. Create a free account at [Cloudfare](https://dash.cloudflare.com)
   
2. Upon joining, Cloudfare should be asking you to add a site. If you already had an account, there's a link for adding a site somewhere on the dashboard. Add the domain you acquired previously.

3. Don't do any of the paid stuff (unless you want to, ofc).
   
4. Once your domain is ready, click on it on your dashboard, and click on *DNS* in the left toolbar.
   
5. We're going to add an ```A Record``` here, click the *Add Record* button
    - Type: ```A```
    - Name (required):```<your-snazzy-domain>.<something>```
        - eg: ```example.com```
    - IPv4 address: ```<Your.Server.IP.Address>```
    - Proxy status: *Disabled*
    - TTL: *Auto*

6. That's it for now! We'll come back here later for a couple of things:
    - Generate an API Token to update our domains and aliases with Let's Encrypt certificates **automatically**
        - aka we don't have to remember when our ssl certeficates are expiring for each of our apps
        - AND each chart can get it's own automatically updated and validate certificate
        - we'll do that automatically with a Trueharts app we'll install on our homelab!
    - Add a ```CNAME Record``` for each app we want to access remotely
        - eg if we wanted to access the Whoogle chart, we would add a CNAME record for whoogle.your-snazzy-domain.something
        - once we set everything up, you should be able to access whoogle.your-snazzy-domain.com from anywhere with your credentials
     
Okay! Break time over! Let's get back to it!         

-----

## Part V: App Cluster Network and Remote Access Set Up

-----

Now that our apps cluster is up and running, and we have a shiny new domain name with a DNS service ready to resove our domain, let's set up our network access.

First, we want to change the default ports of the TrueNAS Web GUI (80 for http, 443 for https).

Then, we'll set up a reverse proxy (```traefik```) to use those ports instead to handle forwarding domain aliases (eg coolapp.shinyserver.com) to the apps in our cluster.

We'll also want to replace TrueNAS's default Loadbalancer, and replace it with ```metallb```. ```metallb``` will allow us to specify IP addresses for each of our apps. 
With TrueNAS's default Loadbalancer, our only choice for IP address when we want to expose ports is to use the static IP you set for the server. 

Next, we'll set up ```clusterissuer```, which will get verified Let's Encrypt certificates for each of our apps, on the fly! 
We'll need an access token from [Cloudflare](https://cloudflare.com/) to give ```clusterissuer``` access to pass the security challenges. 

We'll also want to set up ```cloudflareddns``` to keep Cloudflare up to date with our latest IP address, if our ISP is always changing it. 
We'll need another access token from [Cloudflare](https://cloudflare.com/) to complete the request.

Finally in this sections, we'll set up ```blocky``` as custom local DNS service and ad blocker. 


-----

### Step 1: Change the default ports for the TrueNAS Web GUI

1. Navigate to *System* -> *General Settings*, and click on the *Settings* button in the top-right of the *GUI* widget:

![SystemGeneralScreen](https://www.truenas.com/docs/images/SCALE/SystemSettings/SystemGeneralScreen.png)

2. Change *Web Interface HTTP Port* to ```81``` and *Web Interface HTTPS Port* to ```444``` and save
    - **Why are we doing this?**
    - Port 80 is the default port for external access to your network via the IP address via http, and 443 for https
    - When you type ```http://google.com```, for example, the default end point would be to port ```80```
    - If you typed ```https``` instead, that default end port would be ```443```
    - When someone types ```http://your-awesome-server.xyz``` into a browser, they will be presented with whatever service is available at port 80 on your network
    - If the above link was typed with ```https```, they would be presented with whatever service was available at port 443 on your network
    - With TrueNAS on your network, ***that's the Web GUI!***
    - We are changing that here, and we'll instead let ```traefik``` handle what services are presented when remote clients access those ports
  
The WebUI will now prompt you to restart the web server service, after which you will be automatically redirected to your dashboard through one of the new ports. 

If you are not prompted to restart the web server service, you may restart the machine and manually navigate to the WebUI address followed by one of the new ports 

eg. ```truenas.local:81```.

![SystemGeneralGuiSettings](https://www.truenas.com/docs/images/SCALE/SystemSettings/SystemGeneralGuiSettings.png)

-----

### Step 2: Disable SCALE's default LoadBalancer and install MetalLB instead

See here for a guide from TrueCharts: [TrueCharts: MetalLB Setup Guide](https://truecharts.org/charts/enterprise/metallb-config/setup-guide/)

1. Install ```metallb``` from the TrueCharts Operators train, use the default settings
    -  If you encounter an error, run the below command from terminal and try to install again from the Web GUI:

```
k3s kubectl delete --grace-period 30 --v=4 -k https://github.com/truecharts/manifests/delete
```

2. Once ```metallb``` successfully installs, *then you can install* ```metallb-config```

3. On the install screen, create a new entry under *Configure IP Address Pools Object*:
    - *Name*: Enter a general name for this IP range. Something like apps or charts for this field is fine.
    - *Auto Assign*: if you want MetalLB Services to auto-assign IPs from the configured address pool without needing to specify per app. Recommendation is to keep this checked. You can still specify an IP for apps as needed (see step 3).
  
4. Also on the install screen, create a single entry under *Configure Address Pools*:
    - *Address Pool Entry*: Specify an IP range for MetalLB to assign IPs that is OUTSIDE your current DHCP range on your LAN. For example, if your DHCP range is ```192.168.1.100-192.168.1.255```, then your entry can be any range below ```192.168.1.100```. This entry can also be specified in CIDR format.

![metallb_guide_addresspool_basic](https://truecharts.org/assets/images/metallb_guide_addresspool_basic-3a4c701591f41a719f49dc2879a662ca.png)

> For users with VLANs or multiple subnets, you may create create additional address pool objects as needed.

5. Also on the install screen, create a new entry under *Configure L2 Advertisements*.
    - *Name*: Enter a basic name for your layer 2 advertisement.
    - *Address Pool Entry*: This should match the name of the address pool created above (not the IP range itself).

![metallb_guide_l2advertisement](https://truecharts.org/assets/images/metallb_guide_l2advertisement-f957198af0975b52e0fa12f98b3c219c.png)

6. Finally, let's disable the Integrated Load Balancer by going to *Apps* -> *Settings*, then click on *Advanced Settings*, then uncheck *Enable Integrated Load Balancer*

> This will trigger a restart of Kubernetes and all apps. After roughly 5-10 minutes, your apps will redeploy using the MetalLB-assigned IP addresses.
    - You can verify that happened by running the command below in terminal:

```
k3s kubectl get svc -A
```

> If you have an IP conflict with a previously assigned address it will show as ```<pending>```. **Rebooting should fix this***

7. That's it! We now have a more functional load balancer for our apps cluster. You can delve more into ```metallb```'s configuration options here: [MetalLB](https://metallb.universe.tf/configuration/)

8. **Why did we do this?**
    - SCALE's integrated Load Balancer doesn't allow us to specify unique IP addresses per apps, like you can with jails in TrueNAS CORE.
    - ```metallb``` gives us this ability
    - Now, when we set up an app, we can select *LoadBalancer* for apps we want to expose ports for, and *actually specify which IP we want the app to use*
    - We can also leave this blank, and per our config settings, ```metallb``` will select an IP from our allowed range

-----

### Step Three: Install Traefik for reverse proxy / ingress support

1. Go to *Apps* in the Web GUI, click on Discover Apps, and install ```traefik``` from the TrueCharts enteerprise train

2. On the *Install* screen, make the following changes under **Networking and Services**:

- **Main Service** - *Loadbalancer IP*: select an IP in the allowable range you picked for ```metallb```
- **TCP Service** - *Loadbalancer IP*: use the same IP you selected above 
- *web Entrypoint Configuration* > Entrypoints port: Change port 9080 to port 80
- *websecure Entrypoint Configuration* > Entrypoints port: Change port 9443 to port 443

3. Click *Install*, and we're done!
    - You can access your dashboard at the link in the *Notes* widget on the *Apps* tab for ```traefik```
    - The link is ```http://Your.Selected.Loadbalancer.IP:9000/dashboard/```
    - We'll come back here later to set up some authentication middleware
  
-----

### Step Four: Install Clusterissuer for automated Let's Encrypt certificates
See [this guide] from TrueCharts for reference

The purpose of this step is to set up ```clusterissuer```. What ```clusterissuer``` will do is request and renew certificates from Let's Encrypt for all of your apps that need them.

For anyone that has had to set up certificates *and renew them* on their own, this makes life much easier. 

Let's get to it!

1. **Set Nameservers for TrueNAS Server**
    - It is important to configure Scale with reliable nameserver to avoid issues handling DNS-01 challenges.
    - Under *Network* -> *Global Configuration*-> *Nameservers*, TrueCharts recommends setting ```1.1.1.1/1.0.0.1``` (*Cloudfare*) or ```8.8.8.8/8.8.4.4``` (*Google*).
  
![scale-network-nameserver](https://truecharts.org/assets/images/scale-network-nameserver-1f209b68ab798f9d7ab5c970c03d7619.png)

2. **Create a Cloudfare API Token for issuing Let's Encrypt certificates**
    - Go to the [Cloudfare API Tokens](https://dash.cloudflare.com/profile/api-tokens) Dashboard
    - Click *Create Token*
    - Select *Edit Zone DNS template*.

![cf-apitokens-template](https://truecharts.org/assets/images/cf-apitokens-template-d2dbad789d3620a820755b23d0ac3ceb.png)

The recommended API Token permissions from TrueCharts are below:

![cf-apitokens-perms](https://truecharts.org/assets/images/cf-apitokens-perms-cd6fbf348c5782343cc757a76c719ac8.png)

- Save these settings, and Cloudfare will flash an API Token at you. Save this! We need it for the very next step.

3. **Install** ```clusterissuer```
    - Find ```clusterissuer``` in the TrueCharts enterprise train
    - In settings, make the following updates:
    - *Name*: Name this "cloudflareprod". This name will be used later in the app ingress configuration
        - It's possible to set up multiple DNS providers per app, we're setting up one for Cloudfare
        - Feel free to set up some of your own if you choose! 
    - *Type of DNS Provider*: Cloudflare
    - *Server*: Letsencrypt-Production
    - *Email*: The email address you register with Let's Encrypt for renewal/expiration notices
    - *Cloudflare API key*: Leave blank since API token will be used
    - *Cloudflare API Token*: Populate with token created from above.

![clusterissuer-appconfig](https://truecharts.org/assets/images/clusterissuer-appconfig-55e10076d3dec9b4f73f244a60a00e9a.png)

4. That's it! You should now be able to add "cloudfareprod" to any app installation to get auto-generated Let's Encrypt certificates.
    - See below for an example. You would replace ```cert``` with ```cloudfareprod``` if you've been following this guide
  
![clusterissuer-ingressconfig](https://truecharts.org/assets/images/clusterissuer-ingressconfig-cc09b0d1c68d0589d105f728f2521626.png) 

> Note!
> As part of the DNS verification process cert-manager will connect to authoritative nameservers to validate the DNS ACME entry. Any firewall or router rules blocking or modifying DNS traffic will cause this process to fail and prevent the issuance of certificates. Ensure no firewall or router rules are in place blocking or modifying DNS traffic to assigned authoritative nameservers. Below is an example of cloudflare assigned authoritative nameservers (these nameservers are unique to each user).

![cloudflare-nameservers](https://truecharts.org/assets/images/cloudflare-nameservers-6ac9cbf04b84c2f1c14c0be4dcd56ff4.png)

> Anoter Note!
> It is by design that the app does not run, there are no events, no logs and no shell.

![clusterissuer2](https://truecharts.org/assets/images/clusterissuer2-6bc697bc1e3fdd5a38a69dd0e665d9ec.png)

-----

### Step 5: Install Cloudflareddns
See [this guide](https://truecharts.org/charts/stable/cloudflareddns/setup-guide) from TrueCharts for the complete ```cloudflareddns``` tutorial

Next, we're going to install ```cloudflareddns```. Note that this isn't necessary if your IP address from your ISP never changes.

If it does, ```cloudflareddns``` will let Cloudflare know on a regular basis what your IP address is, so whatever it changes to is always tied to your domain.



2. Let's go back to the [Cloudfare API Tokens](https://dash.cloudflare.com/profile/api-tokens) Dashboard
    - Click *Create Token*
    - Select *Edit Zone DNS template*.
    - Let's call it ```cloudflareddns token``` and use the settings below:

![cloudflare-token](https://truecharts.org/assets/images/cloudflare-token-ce3789f4134c14ef0cbd2e96e091682e.png)

3. Make sure to save the resulting API token for the next step!

2. Install ```cloudflareddns``` from the TrueCharts stable train

3. On the Install screen, under *Cloudflareddns Configuration*, set *CF API Token* (not key!) to be the API Token you just generated
    - Everything else can remain as default here
  
4. Also on the Install screen, under *Hosts, Zones and Record Types*, add the following settings:

    - *Domain*: set to your DNS Zone A record (```your-cool-server.xyz```)
    - *Zone*: set to ```DNS Zone ID``` found on the Cloudflare Overview Page for your domain.
    - *Record Type*: ```A``` if you're only changing your main domain (which you should be if you're following this guide)
  
5. And that's it! Cloudflare should now be regularly notifed of what our IP is, and it should always be attached to our domain, even if our IP address is dynamic. 

-----


### Step 6: Install Blocky DNS Service
See the original TrueCharts guide here: [TrueCharts - Blocky Setup Guide](https://truecharts.org/charts/enterprise/blocky/setup-guide/)

1. First, let's install *Blocky* from the TrueCharts enterprise train

2. For DNS, we're going to configure DoT and DoH.
    - What's that? You have no idea what those acronyms mean? That's cool, here's an [article](https://www.cloudns.net/blog/understanding-dot-and-doh-dns-over-tls-vs-dns-over-https/) for ya.
    - DoT and DoH are secure, so we're going to configure those specifically

3. On the Install Screen, under **App Configuration** check *Override Default Upstreams* add each of these under *Upstream Groups*:
```
tcp-tls:1.1.1.1:853
```
```
tcp-tls:1.0.0.1:853
```
```
https://1.1.1.1/dns-query
```
```
https://1.0.0.1/dns-query
```

5. Also on the Install Screen, under **App Configuration**, click add, and name the group whatever you want (I call it ```stevenblack-hagezi```), then add the follwing in that group:
```
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
```
```
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/domains/pro.txt
```
  
6. Also on the Install Screen, under **App Configuration**, add your domain name (eg ```your-cool-server.org```) under *k8s-Gateway Configuration*
    - This allows us to enter our domain aliases on the local network (```neatapp.my-cool-server.xyz```), and Blocky will map those to local IP addresses for us
    - This way, we're not making requests to remote DNS servers to access a PC we're sitting 30 feet from

8. Also on the Install Screen, under **Networking and Services**, for *DNS TCP Service*, *DNS UDP Service*, and *DoT Service*:
    - Set the *Service Type* to **Loadbalancer (Expose Ports)
    - Set the *Loadbalancer IP* to an IP in the allowable range you set up in ```metallb```
    - You can leave the main service set to Cluster IP. We don't need that port exposed, and we'll be able to access blocky later via your domain
> Make sure you set all 3 of the Loadbalancer services to use the same IP address!

7. Scroll down to the bottom an click *Install*

8. Next, we'll want to forward some ports on our router:
   - Forward port ```53``` for the Loadbalancer IP address you set above (Both TCP and UDP)
   - Forward port ```853``` for the Loadbalancer IP address you set above (TCP only)
  
9. And that's it! You now have Blocky acting as your local DNS proxy AND ad blocker!
    - You should be able to set Blocky as your local DNS server on your router if you wish
    - Every router is differnt in this regard, feel free to do an internet search for instructions
    - Basically, the IP address you set for Blocky would be the IP address you use in your router as your local DNS
  
-----

### Step 7: Install ```protonmail-bridge``` (Optional)
Based on these [installation notes for ```protonmail-bridge``` from TrueCharts](https://truecharts.org/charts/stable/protonmail-bridge/installation_notes)

Finally in this section, we're going to set up an email service to allow our server to communicate with us when something happens.

TrueNAS itself will want to send you email notifications whenever something happens, like a cron job completes or if theres a HDD error of some sort. 

You CAN use Google, but they are creepy and mine all of our email for data. 

Proton Mail is super encrypted and secure, Proton couldn't mine our emails for data even if they wanted to, which is great.

If you have a **paid Proton account**, (which I highly recommend, if for nothing else than to support services that provide data privacy to users),
you can use a CLI app called **Proton Mail Bridge**, which allows you to access Proton emails via SMTP. 

This bridge is, well it can be a bit finicky. For example, every now and then, if you stop receiving emails or if your apps throw SMTP errors, it
probably means you need to log into the ```protonmailbridge``` app via shell and login to your proton account again. 

But, once you get it set up, you can have all server mail forwarded to your Proton account, AND you can create different email aliases for different apps, functions, or services!
- Careful with the email aliases! I think you only get ten, and you can't delete them once they are created!

If this all seems too intimidating, you can always use a Gmail account. Gmail has a feature which allows you to generate a unique *app password* for each app that wants access to your Gmail. 
Gmail only shows you this once, so they kind of function like the API tokens we were generating with Cloudflare. 

We'll try and provide instructions for both methodologies. By the end of this section, you should be able to receive emails from your apps and servers, either using Proton or Gmail. 

1. **Install ```protonmail-bridge```**

- In the Install Screen, under **Networking and Services**:
    - change the *Loadbalancer IP* for both *IMAP* and *SMTP* to be an allowable and available IP address in the range you defined for ```metallb```.
    - change the *SMTP Port* to be ```1025```
    - change the *IMAP Port* to be ```1143```
- Scroll to the bottom of the Install screen and click *Install*

2. **Shell into the ```protonmail-bridge``` container using ```heavyscript```**
- First, you want to open you SSH terminal into your TrueNAS server as your admin account, then run this command:
```
heavyscript pod --shell protonmail-bridge
```

- You should see a prompt asking:
> App Name:  protonmail-bridge
> Pod:       protonmail-bridge-5b64468bc5-f6nsq
> Container: protonmail-bridge
>
> If everything looks correct press enter/spacebar, or press ctrl+c to exit

- Go ahead and press ```Enter``` or ```Spacebar```!
- We are now running a terminal *inside of the ```protonmail-bridge``` pod*

3. **Per the installation notes, once shelled into ```protonmail-bridge```, run the following commands in order:
```
chmod +x entrypoint.sh && ps aux | grep [b]ridge | awk '{print $2}' | xargs -I {} kill -9 {} && ./entrypoint.sh init
```
- At this point, you should see some cool ASCII art of a bridge pop up in the terminal, followed by a prompt. Go ahead and enter the following command:
```
login
```

4. **Enter your _Proton Mail_ username and password when asked**
- If you set up a *Mailbox Password* or *2FA*, you'll be asked for those as well.
- If you logged in correctly you should see that:
> A sync has begun for \<your-proton-username\>
- This sync might take a few minutes

5. **Once the sync is complete, let's change some settings:**


- First, enter the following command to allow bridge to securely connect to Proton via 3rd parties:
```
proxy allow
```
-You should see something like this:
> Bridge is currently set to NOT use alternative routing to connect to Proton if it is being blocked.
> Are you sure you want to allow bridge to do this?
- Type ```yes``` and press ```Enter```

- If you use multiple email aliases with Proton (I recommend this), you want to turn on *Split Address Mode*
- Split Address Mode will treat each of your email addresses like a separate SMTP account
- IF that sounds cool, enter the following command into the ```protonmail-bridge``` shell:
```
change mode
```
```protonmail-bridge``` will ask you the following:
> Are you sure you want to change the mode for account \<username\> to split? yes/no:

- Type ```yes``` and press ```Enter```, and ```protonmail-bridge``` will start syncing ***again*** (lol yeah, I know)

6. Let's take a 10 minute break and think about all of those server emails we'll receive in the future 💭📨

7. **When that's done syncing, let's grab our credentials**
- Type the following command into the ```protonmail-bridge``` shell:
```
list
```

- You should see an output like this:
> # : account                     (status         , address mode   )
> 0 : \<your-proton-username\>    (connected      , combined       )

- Take note of the number next to the account we want credentials for, (```0``` in the example above)
- We'll use that number to run the ```info``` command:
```
info 0
```
- grab the password for the username / email alias you want to use for TrueNAS system emails


9. **Set up some system emails for TrueNAS**
- In the top toolbar of the TrueNAS GUI, all the way in the top right:
- click the alert 🔔 icon,
- then click the ⚙️ icon,
- then click *Email*
> You can also get here by going to the *System* tab -> *General* -> Locate the Email Widget

![EmailWidget](https://www.truenas.com/docs/images/SCALE/Dashboard/EmailWidget.png)

- Click *Settings* in the email widget

- In the screen below, make sure to update the following:
    - *From Email*: Your Proton email address
    - *From Name*: Whatever you want
    - *Outgoing Mail Server*: The Loadbalancer IP address you set for ```protonmail-bridge``` above
    - *Mail Server Port*: ```1025```, as we set in the ```protonmail-bridge``` settings
    - *Security*: *TLS (STARTTLS)*
    - *SMTP Authentication*: Checked (enabled)
    - *Username*: Your Proton email address
    - *Password*: The password you copied running the ```info 0``` command  

- Click *Send Test Email* below to test your settings.
- If they worked, you should see that the email was sent successfully, and shortly after, you should receive an email to your Proton account!
- Go ahead and save these settings, and pat yourself on the back for setting up secure email alerts and supporting services that respect user privacy!
  
![EmailOptionsSMTP](https://www.truenas.com/docs/images/SCALE/SystemSettings/EmailOptionsSMTP.png)


7. **Use ```protonmail-bridge```!
- See below for some useful commands
- Here's [a full list of commands from *Proton*](https://proton.me/support/bridge-cli-guide)

To terminal into ```protonmail-bridge```, run this ***from TrueNAS shell or SSH terminal***:
```
heavyscript pod --shell protonmail-bridge
```

In ```protonmail-bridge``` shell, run this to launch the ```protonmail-bridge``` terminal:
```
chmod +x entrypoint.sh && ps aux | grep [b]ridge | awk '{print $2}' | xargs -I {} kill -9 {} && ./entrypoint.sh init
```
- If you do this a lot, you can press ```Arrow Up``` in the ```protonmail-bridge``` shell and see the command pop up

If you were logged out for whatever reason, run this after the command above:
```
login
```

Run this to get the list and index numbers for each of your Proton accounts:
```
list
```

Run this to get the credentials for all of your email aliases, where ```n``` is the index number for your account from the command above:
```
info n
```

-----

### Step 8: Use Google for System Emails Instead (Discouraged)


-----

## Part V: App Cluster Single Sign On and Authentication Set Up

-----

See these guides from [TrueCharts]() for the details behind this tutorial:
- [Authelia + Lldap + Traefik Forwardauth Setup Guide](https://truecharts.org/charts/enterprise/authelia/Setup-Guide/)
- [Lldap Installation Notes](https://truecharts.org/charts/stable/lldap/installation-notes/)

Okay, by now, we should have a shiny domain for our server that's remotely accessible from anywhere. 
Welllll, that's really cool, but what about all of the shadowy figures in hoods and masks with curly mustaches that want to steal all of your electronic valuables??

You know the type...

![Tenor - Hacks Hacker GIF](https://tenor.com/bBR6R.gif)

That's why we're going to set up ***Single Sign On***.

Wuzzat even??

The idea here is that, let's say we have a bunch of apps. Like ```gitea``` (open source alternative to sites like *github.com*)  and ```guacamole``` (web-based remote desktop service)

Neat! Now let's say you have two friends, ```alice``` and ```bob```, who both need access to both ```gitea``` and ```guacamole```. 

Wellll, each app has their own internal username and password system, so we would need to set up an account for ```alice``` and ```bob``` ***in each app***.

We also need to set up a default password for them ***in each app***.

It would be up to your users to keep track of which password is being used for which account.

And, well, people are busy, and they have a lot of darn passwords to remember.

So what happens? **They will pick a super insecure password that's easy for them to remember, _and also easy for hackers to guess and gain access into your server_**.

And now you have a security risk.

With ***Single Sign On***, your users will only need to remember (mostly) one set of usernames and passwords. 
We will use an **LDAP Server** (```lldap```), to be our single source of truth for usernames and passwords.
When users want to update their password, they will go to one place, let's say ```lldap.your-cool-server.xyz```. 
```gitea``` and ```guacamole```, once set up to do so, will look to ```lldap``` for the correct usernames and passwords instead of their own internal user databases. 
Now that gaining access to your system and changing passwords is easier for your users, you can count on them to create and remember strong passwords to enter your server.

For authentication, we're going to use ```authelia``` which is an authentication middleware. The idea is that, whenever anyone wants to access anything at our domain,
```authelia``` will chime in to ask for a username and password if it hasn't received one from that client node. ```authelia``` will check for the correct
usernames and passwords in ```lldap```, the same place your other apps are checking. 

If ```authelia``` finds a valid account in ```lldap```, AND that account is part of an ldap group that has permission to access the site the user is trying to go to,
**only then will** ```authelia``` **grant access to the app requested**.


By the time we make it to the end of this section, we should have a framework for assigning different levels of access to apps based on group membership, 
and a single source of truth for usernames and passwords for most or possibly all of your apps!


-----

### Step One: Install and Configure ```lldap```
See here for the [relevant guide from TrueCharts](https://truecharts.org/charts/stable/lldap/installation-notes/).

1. Install ```lldap``` from the TrueCharts stable train

2. In the Install screen under **Workload Settings**, change the following:
    - *LDAP Base DN*: if your server's domain name is ```my-cool-server.org```, you would enter ```dc=my-cool-server,dc=com``` here
    - *Ldap User DN*: create an admin username to log into lldap with
    - *Ldap User Email*: associate an email address with this admin account
    - *Ldap User Password*: set a password for this admin account
    - *Public URL*: we're going to give this app an alias to access it with soon, for now, choose something like ```https://lldap.<your-shiny-server>.<com>```
        - Replace ```<your-shiny-server>``` with your domain, and ```<com>``` with your top-level domain
    
3. In the Install screen under **Workload Settings**, check *Show SMTP Settings* and a bunch of options should pop up
    - If you did NOT set up ```protonmail-bridge```, use the Google SMTP instructions to fill the below out. Otherwise:
   -*SMTP Server Url*: ```http://<Your.Protonmail-bridge.IP.Address>```, as we set in ```protonmail-bridge```
   -*SMTP Port*: ```1025```, as we set in ```protonmail-bridge```
    - *SMTP Encryption*: ```STARTTLS```
    - *SMTP User*: Your Proton email address
    - *SMTP Password*: Your Proton email password, the ones you get from shelling into ```protonmail-bridge```, logging in using the instructions in this guide, and copying the password for your email address
    - *SMTP From*: ```LLDAP Admin <sender@protonmail.com>```

3. In the Install screen under **Network and Services**, for the **Main Service** only, change the following settings:
    -  *Service Type*: ```Loadbalancer (Expose Ports)```
    -  *LoadBalancer IP*: An available IP address from your allowed range in ```metallb```

4. In the Install screen under **Network and Services**, for the **Additional service to accept LDAP connections** only, change the following settings:
    -  *Service Type*: ```ClusterIP (Do Not Expose Ports)```
    -  **Why are we doing this??**
    -  Welll, we want a defined IP address and available port for the main service, which is the ```lldap``` web GUI
    -  We don't need that for the LDAP connection service, so we leave those ports unexposed, and the IP address undefined
    -  I think it's better to expose as few ports as possible, generally speaking.

5. Go ahead and click Install at the bottom of the Install screen.
    - Well come back here later to set up Ingress after ```authelia``` is all set up

6. In your web browser, enter ```http://<Loadbalancer.IP.For.Lldap:17170```
    - Log in using the Lldap username and password you set above

7. Once logged into ```lldap```, let's create another user that our apps will use to bind credentials to the LDAP server
    - In the *User* screen, click *Create a User*
    - In this example, we'll call them ```authelia```
    - You need to fill out an email for the user (let's use the same email you use for TrueNAS alerts)
    - Use a strong password (consider a password generator) and save it in a password manager
    - Once the user is created, you'll be returned to the *User* screen
    - Scroll to the bottom, and add this user to the ```lldap_password_manager``` group
  
That's it! Lldap is all set up! We'll come back to update the config in a bit once we've finished setting up ```authelia```

-----

### Step 2: Install and Configure ```authelia```
Based on [this guide](https://truecharts.org/charts/enterprise/authelia/Setup-Guide#setup-authelia) from TrueCharts

> The setup for Authelia is very specific, and the logs won't tell you where you've messed up,
> but there's precise steps used to integrate LLDAP into Authelia.
> The info comes from the LLDAP Authelia Docs and the upstream repo.

1. Install ```authelia``` from the TrueCharts enterprise train to bring up the Install screen

2. On the Install screen, under **App Configuration**, update the following settings:
    
    - *Domain*: ```mydomain.com``` - Your domain without https://
    - *Default Redirection URL*: ```https://auth.mydomain.com``` - Can be anything, but we'll stick to auth.mydomain.com. As well, this will be the ingress URL for Authelia
  
3. On the Install screen, also under **App Configuration**, make sure **LDAP backend configuration** is checked, then updated the following settings:

- *Implementation*: Custom (that's the default)
- *URL*:
```
ldap://lldap-ldap.ix-lldap.svc.cluster.local:3890
```
- *Connection Timeout*: ```5s```
- *Start TLS*: (Not necessary)
- *TLS Settings*: (Not necessary)
- *Server Name*: Leave blank
- *Skip Certificate Verification*: Leave unchecked
- *Minimum TLS version*: TLS1.2
- *Base DN*:
```
dc=mydomain,dc=com
```
- *Username Attribute*:
```
uid
```
- *Additional Users DN*:
```
ou=people
```
- *Users Filter*:
```
(&(|({username_attribute}={input})({mail_attribute}={input}))(objectClass=person))
```
- *Additional Groups DN*:
```
ou=groups
```
- *Groups Filter*:
```
(member={dn})
```
- *Group name Attribute*:
```
cn
```
- *Mail Attribute*:
```
mail
```
- *Display Name Attribute*:
```
displayName
```
- *Admin User*:
```
uid=authelia,ou=people,dc=mydomain,dc=com
```
- Notice the ```uid=authelia```, this is the admin user we created in ```lldap``` to manage LDAP authentications for us
- *Password*: ```RANDOMPASSWORD```
    - This is the strong password you created for the *authelia* user 

4. On the Install screen, also under **App Configuration**, make sure **SMTP provider** is checked, then updated the following settings:
> If you opted for gmail instead of protonmail, ignore the below and use the Google instructions to generate an app password
- *Host*:
```
Your.Protonmail-bridge.IP.Address
```
- *Port*:
```
1025
```
- *Username*: ```your@protonmail.address```
- *Password*: ```YoUr-RanD0M-PaSsWoRD``` from ```protonmail-bridge``` credentials, acquired running ```info 0``` in the bridge shell
- *Sender*: ```your@protonmail.address```
- *Identifier*: ```127.0.0.1``` - this matches the host ID in ```protonmail-bridge```'s certificate
- *Startup Check Address*: ```your@protonmail.address```
- *Disable Require TLS*: checked
- *Skip Certificate Verification*: checked

5. On the Install screen, under ***Networking and Services**, set *Service Type* to *Cluster IP (Do Not Expose Ports)*

6. On the Install screen, under **Ingress*, click * Enable Ingress*,
    - Then click *Add* next to *Host*,
    - Under *Hostname*, add ```auth.my-cool-server-domain.com```
    - Then click *Add* next to *Path*
    - For *Cert-Manager clusterIssuer*, enter ```cert```
  
7. Scroll to the bottom and click *Install*. ```authelia``` should be running after a couple of minutes!
    - Don't worry, nothing really works yet.
    - We are going to finish setting up ```authelia``` in the next section
  
-----

### Step 3: ```authelia``` - Set up Access Control Restrictions
Many of these rules are based on [this guide on Authelia Rules](https://truecharts.org/charts/enterprise/authelia/authelia-rules) from TrueCharts

Access Control Restrictions?? What are thooooseee???

This is where we'll set up access tiers for our users. 
Say, for example, you have many apps, and you want to give ```alice``` and ```bob``` access to only a few of those. 
We can use ```lldap``` groups and ```authelia``` access control restrictions to do that!

Access control restrictions is a complex topic. See here for [Authelia's documentation on the many ways you can set this up](https://www.authelia.com/configuration/security/access-control/)

> **The order of the rules is significant!!**
> The rules are each evaluated by ```authelia``` **in order**, so it stops evaluating rules after the first match.
> **It's important to define these rules in order!**
> 
> IF there's a rule you think you might want to use later, it might be best to add it now.
> If you don't have the domain set up, that' okay. The rule won't do anything until you do,
> and you can easily update the config later if you want to change the domain list
>
> If you don't add a rule now but want it later, **you'll likely have to delete all of your rules and start fresh**

Here, we're going to do two setups. The first one will be what I would call a 'simple' setup. Only one admin tier, which has access to all sites, and deny everything else:

***The Simple Setup***

![AutheliaAccessControl](https://truecharts.org/assets/images/AutheliaAccessControl-54b4a1b78f797c792cd493c8baedff08.png)

- Let's break down what's going on here. Understanding this simple example will help you understand the more complex setup we'll introduce next.
- *Default Policy*: This is what applies before any other rules we set up. ```one-factor``` means a password is required to proceed
- *Networks*: You can define separate networks with ranges of IP addresses, for use in the Rules below
- *Rules*: Rules are constructed in groups, each consisting of:
    - Domain strings, which can take wildcards
    - An authentication policy: ```deny```, ```bypass```, ```one-factor```, and ```two-factor```
    - A Subject: these are the groups for which this authentication policy will apply for the domains listed (you can add as many as you like)
    - Networks: the networks these rules should apply to, defined above (helpful for complex networks)
    - Resources: regex strings which define strings *after the domain name* (helpful for allowing API calls)



***The Advanced Setup***
*First Rule*: Allow API Calls
This rule should allow any call to a domain with api tags to bypass authentication. That way, you don't have to log into ```authelia``` when 
accessing a server app from something like a phone app. 

- *Domain*:
```
*.your-cool-server.com
```
- *Policy*: ```bypass```
- *Subject*: Not Used (Do Not Add)
- *Networks*: Not Used (Do Not Add)

- *Resources*:
```
^/api([/?].*)?$
```
```
 ^/identity.*$
```
```
^/triggers.*$
```
```
^/meshagents.*$
```
```
^/meshsettings.*$
```
```
^/agent.*$
```
```
^/control.*$
```
```
^/meshrelay.*$
```
```
^/wl.*$
```

![authelia-api](https://truecharts.org/assets/images/authelia-api-b448c96f6496784e11be7bc288b53060.png)

*Second Rule*: Valutwarden - Protect Admin Page
What is [Vaultwarden](https://github.com/dani-garcia/vaultwarden)? It's essentially a lightweight, open source, self-hosted [Bitwarden](https://bitwarden.com/) server.
It's one of the many apps available in the [TrueCharts](https://truecharts.org/) ecosystem.

These rules will protect the Vaultwarden admin page with Authelia but bypass when accessing the web vault. 
> The order of these rules is critical or the admin page will not be protected.

- *Domain*: 
```
vaultwarden.your-cool-server.com
```
- *Policy*: ```one_factor```
- *Subject*: Not Used (Do Not Add)
- *Networks*: Not Used (Do Not Add)
- *Resources*:
```
^*/admin.*$
```

![authelia-vw1](https://truecharts.org/assets/images/authelia-vw1-704ba631444d8bded6b5994917d78bc6.png)

*Third Rule*: Valutwarden - bypass for web vault access

- *Domain*: 
```
vaultwarden.your-cool-server.com
```
- *Policy*: ```bypass```
- *Subject*: Not Used (Do Not Add)
- *Networks*: Not Used (Do Not Add)
- *Resources*: Not Used (Do Not Add)

*Fourth Rule*: Domains to bypass

This section will be for domains that are public, aka anyone can access them without a password.
Some potential example of apps you might want to do this with are apps like:
- ```whoogle``` - self-hosted meta search engine
- ```libreddit``` - self-hosted reddit gui

- *Domain*: add any domains with aliases that you want to bypass auth for here, eg:
```
whoogle.your-cool-server.com
```
```
libreddit.your-cool-server.com
```
- *Policy*: ```bypass```
- *Subject*: Not Used (Do Not Add)
- *Networks*: Not Used (Do Not Add)
- *Resources*: Not Used (Do Not Add)

*Fifth Rule*: Access for users to specific services

This rule is for ```lldap``` groups of users that you want to grant access to certain services.
Simply create a group in ```lldap``` and add users to it.
You would add that group in the *Subject*, and any *Domains* they need access to. 

> For multiple tiers of users, you would add rules addtional groups after this rule
> The final rule in this guide should always be the last rule

- *Domain*: add any domains with aliases that you want to require auth for here, eg:
```
sonarr.your-cool-server.com
```
```
radarr.your-cool-server.com
```
- *Policy*: ```one_factor```
- *Subject*:
```
group:your_new_lldap_group
```
- *Networks*: Not Used (Do Not Add)
- *Resources*: Not Used (Do Not Add)

![authelia-user](https://truecharts.org/assets/images/authelia-user-5a9eca70e0884b1fe1bb3d23d7f01ed9.png)

*Final Rule* - Catch-all for the admin users

This rule will catch any access requests not covered by other rules.

All domains not otherwised specified will require admin access.

> However many rules you end up with, **make sure this is the final rule!**


- *Domain*: these two strings together will cover all calls to your domain
```
your-cool-server.com
```
```
*.your-cool-server.com
```
- *Policy*: ```one_factor```
- *Subject*: - only admin accounts should have this level of access
```
group:lldap_admin
```
- *Networks*: Not Used (Do Not Add)
- *Resources*: Not Used (Do Not Add)

![authelia-catch](https://truecharts.org/assets/images/authelia-catch-5bbeea760aebbf3a76547ebd99dc2fe0.png)

-----

### Step 4: ```traefik``` - Set up Forward Auth

Okay! Almost there! Now, we're going to set up ```authelia``` as an authentication middleware in ```traefik```.

1. Go to the *Apps* tab in the TrueNAS Web GUI, click on the ```traefik``` app, and click *Edit* in the *Application Info* widget.

2. Scroll down to *forwardAuth* and click *Add*

- Name your forwardauth something you'll remember, since that's the middleware you'll add to your ingress going forward. Most people use ```auth```
- *Address*:
```
http://authelia.ix-authelia.svc.cluster.local:9091/api/verify?rd=https://auth.your-cool-server.com/
```
- and replace the last part based on ```your-cool-server.com```, and if you've changed ports/names you can get that from Heavyscript
- Check trustForwardHeader
- Add the following authResponseHeaders (press Add 4 times)
```
Remote-User
```
```
Remote-Group
```
```
Remote-Name
```
```
Remote-Email
```

![TraefikForwardAuth](https://truecharts.org/assets/images/TraefikForwardAuth-ac2d65daffb74fdd68b384c939110749.png)

-----

### Step 5: Using ```authelia``` and ```clusterissuer``` to add security to your new apps

Okay! Now that we did all of this setup work, let's watch it pay off! 

Now, whenever we install a new app, we can easily and automatically add a reverse proxy, authentication, and a Let's Encrypt certificate!

The process looks something like this:
- Click on the app in the *Apps* tab of the TrueNAS Web GUI, then click on *Edit* in the *Application Info* widget
- Scroll down to the **Ingress** section, and check *Enable Ingress*
- Under *Hosts*, add the alias for your app (eg: ```sonarr.your-cool-server.com```)
- Click *Add* next to *Paths*
    - *Path*: ```/```
    - *Path Type*: ```Prefix```
- *Cert-Manager clusterIssuer*: ```cloudfareprod```, or whatever you defined for ```certissuer```
- *Traefik Middlewares*: click *Add*, then type ```auth```, or however you defined the *forwardAuth* in ```traefik```
- **Finally, log into [Cloudfare](https://dash.cloudflare.com/), click on your domain in your dashboard, then click on *DNS* -> *Records* on the right menu bar
    - Click *Add Record*
    - *Type*: ```CNAME```
    - *Name*: some alias, eg if your alias is ```sonarr.your-cool-server.com```, then enter ```sonarr``` here
    - *Target*: ```your-cool-server.com```
    - *Proxy status*: ```DNS Only``` (unchecked)
    - *TTL*: ```Auto```
    - Click *Save*
 
> Make sure you create a CNAME record for every alias you use!
> For example, we'll need one for ```auth.your-cool-server.com``` at least
> You'll also likely want one for ```lldap.your-cool-server.com``` so that users can remotely change their passwords
 
At this point, you could go back to edit the configs of the following apps we've installed, and add Ingress to them if you want:

- ```traefik```
- ```blocky```
- ```lldap```
- ```traefik```

![TraefikForwardAuthMiddleware](https://truecharts.org/assets/images/TraefikForwardAuthMiddleware-77182ca6fc6e619540467b0aef732ad2.png)

Great job! You really have accomplished a lot if you made it this far!

Take a break, and enjoy logging into, well, not much yet, but we can change that soon! Great job!

-----

-----

## Appendix - Troubleshooting

-----

### Upgrade from Blufin to Cobia - Issues with SMB permissions

I'm documenting this bug because it's a weird one, and as far as I can tell, I was the only one experiencing it.

SO, I don't know what could be prodcing this except that I upgraded from Bluefin to Cobia, *after* having followed [this guide from TrueCharts](https://truecharts.org/manual/SCALE/guides/dataset/#smb-access)

The problem is, in Cobia, there is no **Auxilliary Parameters** setting for SMB shares, so you can't `force user = apps` or `force group = apps`. 

As far as setting this for your shares so you can follow the recommended TrueCharts setup (yes, this is still recommended as of Cobia, I checked with the TrueCharts developers on Discord),
you can use [this script](https://github.com/xstar97/scale-scripts#smb-auxillary-param) from [Xstar97TheNoob](https://github.com/xstar97) to force user and group for the SMB shares you are also sharing with apps.

The script allows you to choose per share, which is great if you have some shares you **_don't want to share with apps, but want more granular permissions for_**.

The script above **does not** fix the issue we're talking about. 

**If you upgraded from Bluefin to Cobia, and are all of a sudden having read / write / mount permission issues with your smb shares:**
- Run this from an SSH terminal to your server:
```
testparm -s
```
and check if you see `force user = apps` or `force group = apps` anywhere in the options under `[global]`. 

**If you do:**
- First, in the Web GUI, go to the little profile icon in the top right of the screen, click on it, and click **API Keys**. Generate one, and don't lose it! We're going to use it for this specific purpose, then delete it.
- Now, run:
```
curl -X 'GET' \
  'http://localhost:81/api/v2.0/smb' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer <API_KEY>'
```
- Replcae `<API_KEY>` with the API Key you generated above
- If you did **NOT** set up a reverse proxy with `traefik` according to the guide, change the port from `81` to `80`.

**Do you see something like this in the output?**
```
 "smb_options": "force user=apps\nforce group=apps",
```

**If you do:**
- Run this from an SSH terminal to your server (*yeah, the whole thing, just paste it all in there and press `Enter`*):
```
curl -X 'PUT' \
  'http://localhost:81/api/v2.0/smb' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer <API_KEY>' \
  -H 'Content-Type: application/json' \
  -d '{"smb_options":""}'
```
- Replcae `<API_KEY>` with the API Key you generated above
- If you did **NOT** set up a reverse proxy with `traefik` according to the guide, change the port from `81` to `80`.

**That's it!**
- Running `testparm -s` again should not show anything about the apps user or group in your global SMB settings
- You can use [Xstar97TheNoob's script](https://github.com/xstar97) to force user and group to be `apps` per SMB share
    - you would do this for data that you want to share with apps via NFS, but **also want to access via SMB with other users**
 
-----



