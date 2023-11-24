---
layout: default
title: 1. Set up TrueNAS SCALE Bootable USB
parent: I. Setting up the TrueNAS System
grand_parent: TrueCharts Build Tutorial
nav_order: 1
---

# Set up TrueNAS SCALE Bootable USB
1. First, download the installer from here, this should be the latest Cobia build: [Download TrueNAS SCALE][downloadTruenasScale]
2. Next, you'll want to burn this to a thumb drive, using something like [Rufus][rufus] on Windows.
3. If on Windows, use [Rufus][rufus] to burn the iso to the USB and skip to step 8.
4. On Linux, you'll want to follow the instructions below, pulled from here: [Preparing the Install File][prepareInstallFile]
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

----

[downloadTruenasScale]: https://www.truenas.com/download-truenas-scale/
[rufus]: https://rufus.ie/
[prepareInstallFile]: https://www.truenas.com/docs/scale/23.10/gettingstarted/install/installingscale/#preparing-the-install-file