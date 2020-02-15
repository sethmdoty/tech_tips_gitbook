# /dev/sda1 is apparently in use by the system ; will not make a filesystem here!

After installing the drives into the machine, creating the raid, and rebooting something was amiss. 

fdisk -l 

Disk /dev/sda: 1197.9 GB, 1197998080000 bytes 

255 heads, 63 sectors/track, 145648 cylinders 

Units = cylinders of 16065  _512 = 8225280 bytes_ 

_Disk /dev/sdb: 145.9 GB, 145999527936 bytes_ 

_255 heads, 63 sectors/track, 17750 cylinders_ 

_Units = cylinders of 16065_  512 = 8225280 bytes 

Device Boot Start End Blocks Id System 

/dev/sdb1  _1 867 6964146 83 Linux_ 

_/dev/sdb2 868 17495 133564410 83 Linux_ 

_/dev/sdb3 17496 17750 2048287+ 82 Linux swap / Solaris_ 

_Disk /dev/dm-0: 1197.9 GB, 1197998080000 bytes_ 

_255 heads, 63 sectors/track, 145648 cylinders_ 

_Units = cylinders of 16065_  512 = 8225280 bytes

The partition has been created lets create the filesystem. 

mkfs.ext3 /dev/sda 

/dev/sda1 is apparently in use by the system; will not make a filesystem here! 

Of course looking at _mount_ or _fuser /dev/sda1_ resulted in no information the drive wasn’t mounted anywhere or in use. But after many days of working late I finally came back to that /dev/dm-0. Some further google fu resulted in the following two commands. 

dmsetup status 

dmsetup ls 

This showed that there were several mpath devices configured on the system. Since this system is not using software RAID or MPIO this is a problem it should only return 

\# dmsetup ls 

No devices found 

\# dmsetup status 

No devices found

So the final fix this, tell MPIO and Multipath to get the hell out of your system by editing _/etc/multipathd.conf_and adding the following. 1

blacklist { 

devnode '\*' 

}

 Reboot the server, the /dev/dm-0 will be gone. ◦ And you can run mkfs.ext3 /dev/sda1 and then go outside and run in a field.

