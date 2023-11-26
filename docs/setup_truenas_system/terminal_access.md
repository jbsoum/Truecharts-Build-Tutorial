---
layout: default
title: 5. Set up SSH Access
parent: I. TrueNAS System Setup
grand_parent: TrueCharts Build Tutorial
nav_order: 5
---


# Set up SSH Access
- The idea here is that you're unlikely to have a monitor attached to your server at all times
- There is a Shell you can access via the web GUI, but you should not expose that to the internet, and it's a bit finicky
- So, without a monitor, how can you interact with the terminal? What if you're away from your desk?
- Enter SSH! You don't want to open your server up to the internet, but *if we can create a secure tunnel between the server and whatever machine we're on, that should be pretty secure*!
- I understand that SSH is cryptic for those unfamiliar with what it is or how it works, so I'll try to demystify the process

- The idea is that you, the client, wants to generate a set of keys, a *public key* and a *private key*
- The public key is like your address, it lets other people know who you are
- The private key is the secret algorithm that's used to encrypt your connection with whatever device you are connecting to, aka the server

{: .warning }
**NEVER SHARE YOUR PRIVATE KEY WITH ANYONE** 🤫

- See below for a helpful picture from [ssh.com][ssh.com]. You are the client, trying to connect to the server via SSH.

![A visual of how SSH works...you are the client, trying to ssh into the server...][imgSshProtocol]

Let's walk through the steps required to generate a set of host keys for your laptop, or desktop, or whatever device you want to use to connect to your server.
You can set up more than one device, but setting that up is a bit more complex. If you're here, you're probably not ready for all of that yet.
We're going to use OpenSSH, which is available by default on Linux or Mac. 

If you're on Windows, here's some helpful instructions for getting OpenSSH up and running on the command line: [Microsoft - OpenSSH instructions][microsoft-openssh-instructions]
If you can't stand OpenSSH, there is also Putty, if you want, I guess.

1. **Forward a port for your SSH connection on your router**
- This step is different for everyone depending on your router, search for ```Port forwarding <YourRouterNameAndModel>```
- You'll want to forward *some port above 40000*,

{: .why }
> Well, 22 is the default port for SSH connections, and it's a popular one for hackers to try and break into
> We are going to keep that port closed, and open up another random port instead for SSH connections
> This is more secure

- Use TCP only,
- For IP address to forward to, use the static IP address you selected for your server.
- We'll come back to this port forwarding screen for your router to give other apps remote access as needed
  
{:style="counter-reset:none"}
1. **Set up the SSH Service in TrueNAS**
- Go To *System Settings* -> *Services*

![ServicesSCALE][imgServices]

- Make sure the *SSH*, *NFS*, and *SMB* Services are *Running*, and are set to *Start Automatically*
- On the *SSH* line, all the way to the right, click the little pencil icon to Configure SSH.
- Set the TCP port for SSH to be *the port you forwarded on your router in Step 1*.
- Enable *Allow TCP Port Forwarding*
- Do NOT enable *Password Authentication*
    - **Why are we doing this?**
    - We are going to set up access via SSH, so we can keep our system extra secure by making sure the *only way* to access the system remotely is with a valid SSH key registered with the server
- Save these settings

![ServicesSSHBasicSettingsGenOptionsSCALE][imgServicesSSHBasicSettings]

{:style="counter-reset:none"}
1. **_On the client machine (the machine you'll be accessing the server from)_, generate an ssh key with ssh-keygen**
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

{: .note-title}
> ```ssh-keygen``` Output:
>
> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]

- If you leave both of these fields empty, then your SSH key can be used without a password. **I DO NOT RECOMMEND THIS**.
- Lock your key with a secure password. That way, no one can use your key to gain access to your server without that passphrase.

{:style="counter-reset:none"}
1. **Register your key with SSH Agent**
- Run the below command to get ssh-agent running:
```
eval "$(ssh-agent -s)"
```

- You should see something like ```Agent pid 12345``` returned.
- Next, add the key to ssh-agent with the following command:
```
ssh-add /path/to/save/your/private/key
```

{: .warning}
You want to register your **private key** here, not the public one ending in *.pub*

 {:style="counter-reset:none"}    
1. **Still on the client machine, run the below command, which will return your client machine's public key**
```
cat /path/to/save/your/private/key.pub
```
- So, whatever your public key's name is, we are going to return its contents to the terminal screen
{: .warning}
> Do NOT do this with your private key! 🤫
- Copy this by selecting it in the terminal, holding ```CTRL``` + ```SHIFT```, and press ```C```.

{:style="counter-reset:none"}
1. **Register this key with your admin account**
- Note that if you are upgrading from a previous version of FreeNAS / TrueNAS, you're probably using hte ```root``` account as your admin account
- This is being deprecated in future releases, so be prepared to use a separate admin account for all of your root stuff as this guide suggests eventually...
- Go to  *Credentials* -> *Local Users*, and select your admin account for editing

![UserScreenUserDetails][imgUserScreenUserDetails]

- In this screen, we are going to paste the public key we copied in Step 5, and paste it in the *Authorized Keys* section

![AddUserHomeDirAuthSCALE][imgAddUserHomeDirAuth]

- Make sure *SSH Password Login* is **UNCHECKED**
    - **Why are we doing this**?
    - Wellll, we don't want to use passwords to gain SSH access with the server.
    - The only way in should be a secure key known by the server, further secured with a password on the client side.

{:style="counter-reset:none"}
1. **Now, you should be able to log into your server remotely using the command below**:
```
ssh <YourAdminAccount>@<YourServerDomain> -p <YourSSHPort>
```

----

[ssh.com]: https://www.ssh.com
[microsoft-openssh-instructions]: https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui

[imgSshProtocol]: https://www.ssh.com/hubfs/Imported_Blog_Media/SSH_simplified_protocol_diagram-2.png
[imgServices]: https://www.truenas.com/docs/images/SCALE/SystemSettings/ServicesSCALE.png
[imgServicesSSHBasicSettings]: https://www.truenas.com/docs/images/SCALE/SystemSettings/ServicesSSHBasicSettingsGenOptionsSCALE.png
[imgUserScreenUserDetails]: https://www.truenas.com/docs/images/SCALE/Credentials/UserScreenUserDetails.png
[imgAddUserHomeDirAuth]: https://www.truenas.com/docs/images/SCALE/Credentials/AddUserHomeDirAuthSCALE.png