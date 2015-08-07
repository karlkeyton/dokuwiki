
[Source](http://san.ninja/redhatcentos-adding-storage-without-reboot/ "Permalink to RedHat/CentOS/Ubuntu – Adding Storage without Reboot")

# RedHat/CentOS/Ubuntu – Adding Storage without Reboot

[__Mattox Mattox][1] __July 31, 2014

**NOTE:**&nbsp;These steps assumes you have already added/expanded the underlying&nbsp;disk.

### Recognize Storage Change

Now we need to get the guest to see the added storage. Before doing this, you should do an&nbsp;**fdisk -l**&nbsp;to get a list of the current drives and their sizes. This helps in identifying the newly added/extended disk. A new disk&nbsp;_should_&nbsp;show up at the bottom of the list. The commands are slightly different depending on whether you added a new disk or extended an existing one.

    #Added new disk
    echo "- - -" &gt; /sys/class/scsi_host/host0/scan or
    echo "- - -" &gt; /sys/class/scsi_host/host1/scan
    OR
    for i in $(ls /sys/class/scsi_host/); do echo "- - -" &gt; /sys/class/scsi_host/$i/scan; done

    #Extended existing disk
    echo "1" &gt; /sys/class/scsi_device/0:0:X:0/device/rescan
    OR
    for i in $(ls /sys/class/scsi_device/); do echo "1" &gt; /sys/class/scsi_device/"$i"/device/rescan; done

Where&nbsp;**X**&nbsp;is the number of the device you extended. It sometimes takes a couple minutes for a rescan to show up. At the time of writing, I'm not sure how to determine which disk maps to which number, although it appears to go in the order the disks were added.

Now do another&nbsp;**fdisk -l**&nbsp;to confirm your disk is present/changed size.

### Add or Resize PV

If this is a new disk, create a physical volume on the new disk. If you extended an existing disk, resize the physical volume.

    #New disk ONLY
    pvcreate /dev/sd[NewId]
    #Extended disk
    pvresize /dev/sd[Id]
    OR
    pvresize /dev/sd*

### Add or Extend VGs

Add the physical volume to your chosen volume group, usually&nbsp;**appvg**. If you've extended the volume, your volume group should automatically see it.

    #New disk ONLY
    vgextend appvg /dev/sd[NewId]
    #If volume group does not exist:
    vgcreate appvg /dev/sd[NewId];

### Add or Extend LVs

Extend or create your logical volume.

    #Create LV
    lvcreate -L[##][GMK] appvg -n mount_point_lv
    #Extend LV
    lvextend -L+[##][GMK] /dev/appvg/mount_point_lv

Where&nbsp;**##**&nbsp;is the number of units and&nbsp;**GMK**&nbsp;is gigabytes, megabytes, and kilobytes respectively.

### Make or Resize Filesystem

If you've created a new logical volume, you now need to create a filesystem on it.

    #NEW LV ONLY
    #RHEL 6+
    mkfs.ext4 /dev/appvg/mount_point_lv
    #RHEL 5
    mkfs.ext3 /dev/appvg/mount_point_lv

If you've extended a logical volume, you'll need to resize the filesystem so that it can use the added space.

    RHEL 5+ resize2fs /dev/appvg/mount_point_lv

    RHEL 4.8 ext2online /dev/appvg/mount_point_lv

For a newly created Volume you will need to mkdir the target and then add the mount info to /etc/fstab in the same format as other volume groups listed there.

### Create or Resize Swap Filesystem

You have 2 options: Extend swap on an existing logical volume or Create a new swap file

#### Extend Swap

1. Disable swapping for the associated logical volume:

: [root@RHELT01&nbsp;~]# swapoff -v /dev/vg00/swap_lvol

2. Resize the LVM2 logical volume by 500 MB:

: [root@RHELT01&nbsp;~]# lvm lvresize /dev/vg00/swap_lvol -L +500M

3. Format the new swap space

: [root@RHELT01&nbsp;~]# mkswap /dev/vg00/swap_lvol

4. Enable the extended logical volume

: [root@RHELT01&nbsp;~]# swapon –va

5. Test that the logical volume has been extended properly

: [root@RHELT01&nbsp;~]# free -m

#### Create a New Swap Space

Add hard drive

Add new disk to appvg

* echo "- – -" &gt; /sys/class/scsi_host/host0/scan
* pvcreate /dev/sd[NewId]
* vgextend appvg /dev/sd[NewId]
* lvcreate -L 24G -p rw -n swap_lvol appvg

Format new swap space

* mkswap /dev/appvg/swap_lvol

Turn on new swap space

* swapon /dev/appvg/swap_lvol

Turn off old swap space

* swapoff /dev/vg00/swap_lvol

Remove the old swap space logical volume

* lvremove -f /dev/vg00/swap_lvol

Edit /etc/fstab

* echo "/dev/appvg/swap_lvol swap swap defaults 0 0″ &gt;&gt; /etc/fstab
* sed -i ',/dev/vg00/swap_lvol,d' test.txt

Test that swap has been properly configured

![][2]

Storage Guru at Hyatt

I have been working with computers for 10+ years. My specialties are Datacenter Storage, RedHat, and VMWare. I worked for IBM as a Storage Technical Support Specialist for over 2 years. I am currently employed with Caterpillar Inc. where I am a Unix/Vmware system administrator. I graduated Morrison Institute of Technology in Morrison, IL in 2008, where I received a A.A.S. in Systems and Networking Administration; specializing in computer hardware troubleshooting, Windows Server 2008, and Datacenter architecture. Before Morrison Institute of Technology, I attended Sauk Valley Community College in Dixon, IL where I was dual enrolled with Sterling High School in their Computer Information Systems program. I was also enrolled for two years at Whiteside Area Career Center under their IT Systems programs, one of these years was an apprenticeship.

[1]: http://san.ninja/author/mmattox/ "Posts by Mattox Mattox"
[2]: http://media.licdn.com/mpr/mprx/0_JqFQXFQ83oo71M7VJt9MX5cS_EfT0VaVRz1JXkqYy7RKhYJsv-cEEX8KSX7xyjf94cLZIiESLcYR
  
