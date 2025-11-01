# Extend Ubuntu logical volume
Run the following command to find the names of the Logical Volume and Volume Group:<br>
`sudo lvs`

The output should give you the arguments you need for the extend command.<br>
This will be the name of the Logical Volume, and Volume Group.

Run the following command to extend:<br>
`sudo lvextend -l +100%FREE /dev/<vg_name>/<lv_name>`

For example, if the output reads `LV:  ubuntu-lv` and `VG:  ubuntu-vg`, then run the command as:<br>
`sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv`

If you’re using Ubuntu, then the filesystem type is probably `ext4`. Check it just in case:<br>
`df -Th`

...you should find the type of filesystem lined up after the `/dev/mapper/ubuntu---vg-ubuntu--lv

Now that underlying block device is larger, expand the filesystem to use the newly allocated space. The following command is for `ext4`:<br>
`sudo resize2fs /dev/ubuntu-vg/ubuntu-lv`

Verify the results:<br>
`df -h /`

## What happened?
* Physical Disk:  20GB
* Volume Group (ubuntu-vg):  always held the 17GB of space
* Logical Volume (ubuntu-lv):  was only around 9.5GB, is now 17GB
* Filesystem (ext4):  was limited to 9.5GB, now expanded to fill the larger Logical Volume

The default Ubuntu server installation only allocates part of the available space to the root logical volume, leaving the rest unallocated in the volume group for future use. Your `lvextend` command just claimed the unallocated space.

## Why does Ubuntu do this?
Ubuntu uses this LVM/thin provisioning approach for the following reasons:
* Flexibility:  allows you to easily resize volumes later without repartitioning
* Snapshots:  LVM enables easy system snapshots for packups/rollbacks
* Storage Management:  you can easily add more physical disks to the same volume group
* Safety Net:  prevents a runaway process from immediately filling your entire disk

## Would space have run out?
It sure would have!<br>
When you run the `df -h` command, it shows you how much is space is actually available to you. If you fill that storage space up with data, you will get “out of disk space” errors. 

You can check how much available storage is being obscured by LVM/thin with the following command (it will be 0 if you’ve already extended):<br>
`sudo vgs`
