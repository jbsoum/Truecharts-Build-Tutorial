---
layout: default
title: Storage Pool Setup
parent: II. Pool Setup
grand_parent: TrueCharts Build Tutorial
nav_order: 2
---

# Storage Pool 
## where your data will go

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
  
1. **Add the HDDs to your storagepool**
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

{:style="counter-reset:none"}
3. **Review your selection and create your pool**
    - *Pool Name* should be what we set above, (eg ```storage``` as in our example)
    - *Data* under *Topology* should say something like ```<V> x STRIPE| <W> x #.## GiB (SSD)```
        - <V> represents the number of VDEVs in your pool
        - <W> represents the width you selected
    - If everything looks good, go ahead and click *Create Pool*