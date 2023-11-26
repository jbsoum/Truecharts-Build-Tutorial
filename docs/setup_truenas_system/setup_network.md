---
layout: default
title: 4. Set up the Server Network
parent: I. TrueNAS System Setup
grand_parent: TrueCharts Build Tutorial
nav_order: 4
---


# Step Four: Set up the Server Network

1. **Set up the TrueNAS Scale Network**
- Go to *Network*, and for the interface (ethernet port) which is connected to the internet, all the way to the right, click the little pencil icon to Configure

![NetworkInterfacesWidget][imgNetworkInterfacesWidget]

- Under *Aliases*, set the IP Address to your desired local IP address for your server
    - This IP can or cannot be outside of your DHCP range
    - The DHCP range on your router is the range of IP addresses reserved for automatic assignment when someone new connects to the network
    - If you're going to run apps off of this server, you really should have a static IP
    - We'll set it here, in TrueNAS, then set it up on your router as well
- Save these settings
- Still in the *Network* tab of the Web GUI, click on *Settings* to the right of *Global Configuration*

![GlobalConfiguration][imgGlobalConfiguration]

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

![EditGlobalConfiguration][]

{:style="counter-reset:none"}
1. **Set a static IP address for your server on your router**
- This is the IP address you set for the interface above in Step 2
- You typically also need the MAC address for your interface
- You can get that running the command below in the TrueNAS shell under *System Settings*
- You should a bunch of sections, one of them will be labeled with the name of the interface you set an IP address for in Step 2 above.
- In that section, there should be a line that looks something like ```ether ax:3g:8c:04:07:v8```. That string of alphanumeric characters and colons is your MAC address
- You may also need your hostname, which you also set in Step 2 above.
```
ifconfig -a
```

----

[imgNetworkInterfacesWidget]: https://www.truenas.com/docs/images/SCALE/Network/NetworkInterfacesWidget.png
[imgGlobalConfiguration]: https://www.truenas.com/docs/images/SCALE/Network/GlobalConfiguration.png
[imgEditGlobalConfiguration]: https://www.truenas.com/docs/images/SCALE/Network/EditGlobalConfiguration.png