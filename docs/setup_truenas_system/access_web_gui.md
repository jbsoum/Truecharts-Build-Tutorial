---
layout: default
title: 3. Accessing the Web GUI
parent: I. Setting up the TrueNAS System
grand_parent: TrueCharts Build Tutorial
nav_order: 3
---

# Step 3: Accessing the Web GUI
1. After removing the USB and rebooting the system, you should be presented with the Console Setup screen

![ConsoleSetupMenuSCALE][imgConsoleSetupMenu]

{:style="counter-reset:none"}
1. At the top of this screen is the local IP address of the Web GUI

1. From any computer on the same network as your TrueNAS server, you should be able to enter that address and see the Web GUI Login Screen
    - For example, if your Web GUI is at ```https://192.168.1.10:443```, enter that into a web browser on a computer connected to the same network as your server
  
![LoginScreenSCALE][imgLoginScreen]

{:style="counter-reset:none"}
1. Enter the administrator username and password you set up in **Step Two**

1. At this point, you should be logged into the TrueNAS Web GUI!

1. Let's take a quick break, we've accomplished a lot at this point 😌👍🏼

1. *After your break*, check this guide out for a detailed explaination of everything you can do in the Web GUI: [TrueNAS SCALE - UI Reference Guide][imgUiReference]

----

[imgConsoleSetupMenu]: https://www.truenas.com/docs/images/SCALE/CLI/ConsoleSetupMenuSCALE.png
[imgLoginScreen]: https://www.truenas.com/docs/images/SCALE/Login/LoginScreenSCALE.png
[imgUiReference]: https://www.truenas.com/docs/scale/scaleuireference/
[rufus]: https://rufus.ie/
[prepareInstallFile]: https://www.truenas.com/docs/scale/23.10/gettingstarted/install/installingscale/#preparing-the-install-file