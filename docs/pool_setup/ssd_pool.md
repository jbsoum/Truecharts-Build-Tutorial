---
layout: default
title: SSD Pool Setup
parent: II. Pool Setup
grand_parent: TrueCharts Build Tutorial
nav_order: 1
---

# SSD Pool 
## Where your apps will go

Click the **Create Pool** button in the ***Storage*** tab of the TrueNAS GUI.

1. **Select a name for your SSD pool**
- People tend to choose something simple, all lower case without dashes or underscores
- For the purpose of this tutorial, well assume this pool is called ```ssd```
- Leave the drive unencrypted
- Click *Next*

![PoolCreationWizardGeneralInfo](https://www.truenas.com/docs/images/SCALE/Storage/PoolCreationWizardGeneralInfo.png)

{:style="counter-reset:none"}
1. **Add the SSD to your pool**
- For *Layout*, select *Stripe*
- Click *Manual Disk Selection* under *Advanced Options* 
- In the top right corner, click *Add* to add the only VDEV we'll need for this pool
- Select the SSD disk for your pool on the left, and drag it into your new VDEV on the right
- If you have a lot of disks, feel free to use the disk filters to help parse down the list to find your disk
    - We will be setting up backups of our app directory, so disk redundancy isn't super necessary for this pool
    - If you must, you can set up a 2-way or 3-way mirrored VDEV
- Click *Save Selection*, then click *Save and Go To Review*
     
![ManualSelectionScreen](https://www.truenas.com/docs/images/SCALE/Storage/ManualSelectionScreen.png)

{:style="counter-reset:none"}
1. **Review your selection and create your pool**
- *Pool Name* should be what we set above, (eg ```ssd``` as in our example)
- *Data* under *Topology* should say something like ```1 x STRIPE|1 x #.## GiB (SSD)```
- If everything looks good, go ahead and click *Create Pool*

![PoolCreationWizardReviewScreen](https://www.truenas.com/docs/images/SCALE/Storage/PoolCreationWizardReviewScreen.png)
