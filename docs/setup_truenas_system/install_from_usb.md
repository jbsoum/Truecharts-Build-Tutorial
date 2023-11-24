---
layout: default
title: 2. Install from USB & Set up Admin Account
parent: I. Setting up the TrueNAS System
grand_parent: TrueCharts Build Tutorial
nav_order: 2
---

# Install TrueNAS from USB and Set up the Administrator Account

{: .sources }
See here for the guide on first time setup of TrueNAS: [TrueNAS Installer Console Setup][truenasConsoleSetup]

1. Select **Install/Upgrade**

![SCALE Install Main Screen][imgMainScreen]

{:style="counter-reset:none"}
1. Select the desired install drive

![SCALEInstallDriveScreen][imgDriveScreen]
    
- You want to make sure you select the SSD drive you want to use as your boot drive

1. Select **Yes**.

![SCALEInstallWarningScreen][imgWarningScreen]

{:style="counter-reset:none"}
1. Select **Fresh Install** to do a clean install of the downloaded version of TrueNAS SCALE. This erases the contents of the selected drive!

![SCALEInstallUpgradeFresh][imgUpgradeFresh]

{:style="counter-reset:none"}
1. When the operating system device has enough additional space, you can choose to allocate some space for a swap partition to improve performance. Select **Create Swap** and press Enter.

![SCALEInstallPartitionScreen][imgPartitionScreen]

{:style="counter-reset:none"}
1. When existing versions of TrueNAS are present on the device, you can choose **Install in new boot environment** to create a partition or **Format the boot device** to remove previous boot environments.

![SCALEInstallUpdateMethodSelection][imgUpdateMethodSelection]

{:style="counter-reset:none"}
1. Select option **1 Administrative user (admin)** and then **OK** to install SCALE, and create the admin user account. SCALE Bluefin has implemented rootless login. Create an admin account and password. The system retains root as a fallback but it is no longer the default. This account has full control over TrueNAS and is used to log in to the web interface. Set a strong password and protect it. We do not recommend selecting **3 Configure using Web UI**.
![SCALEInstallerConsoleSetupAdminAccount][imgConsoleSetupAdminAccount]
- Next, enter a password for the new admin user.

![SCALEInstallerConsoleSetupAdminPassword][imgConsoleSetupAdminPassword]

{:style="counter-reset:none"}
1. Select Boot via UEFI at the TrueNAS Boot Mode prompt, then select OK and press Enter to begin the installation.

After following the steps to install, reboot the system and remove the install media.

----

[truenasConsoleSetup]: https://www.truenas.com/docs/scale/gettingstarted/install/installingscale/#using-the-truenas-installer-console-setup

[imgMainScreen]: https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallMainScreen.png
[imgDriveScreen]: https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallDriveScreen.png
[imgWarningScreen]: https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallWarningScreen.png
[imgUpgradeFresh]: https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallUpgradeFresh.png
[imgPartitionScreen]: https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallPartitionScreen.png
[imgUpdateMethodSelection]: https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallUpdateMethodSelection.png
[imgConsoleSetupAdminAccount]: https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallerConsoleSetupAdminAccount.png
[imgConsoleSetupAdminPassword]: https://www.truenas.com/docs/images/SCALE/Install/SCALEInstallerConsoleSetupAdminPassword.png