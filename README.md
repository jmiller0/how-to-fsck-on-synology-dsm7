# How to properly do a filesystem check (fsck or e2fck) on Synology DSM 7

Most instructions founds online are for older DSM versions (6).  The following steps were performed by a synology support agent during a support case I created.

I have a DS1821+ and during a recent storm I lost power.  I have a UPS however it failed to shutdown the synology.  
NOTE: Make sure and test your UPS shutdown, if you have many disks it can take longer to shutdown than reserved battery capacity.

After the power down the DSM came back and had a warning on the main volume.

The following steps were obtained by looking at ```/var/log/bash_history.log```.  Before you try any of these steps, attmpt to check the filesystem in Storage Manager, these steps are only if you fail to unmount.

## Here are the steps performed by the synology tech:
| WARNING: Perform hese steps at your own risk! Open a support ticket! |
|----------------------------------------------------------------------|

SSH To your synology and sudo to root:

### Disable all services and unmount volume: This will log out!
```
synospace --stop-all-spaces
```

### Log back in and check that your volume is not mounted, no lvm, mdstat empty
```
mount
lvm lvscan
cat /proc/mdstat
```

### Next we will skip file system check on boot:
```
touch /tmp/volume_skip_check
```
### Next we will disable our volume on startup, replace ```volume1``` with your volume.
```
synosetkeyvalue /etc/synoinfo.conf disable_volumes volume1
```

### Next, go to the DSM GUI and Restart your synology.  
After it comes back online your volume should be unmounted.  Give it a few minutes to finish starting up (5 minutes after ssh is up) and run the commands above to make sure you are not mounted and lvm sees your lvm.  We should be good to fsck now.  
Run this command to see your mapper path to fsck:
```
dmsetup table
```
### Now we'll fsck and auto accept all changes
This can take awhile, you can `nohup <cmd> &` if you want to run in the backgroup, then monitor ```nohup.out```
```
fsck.ext4 -yvf -C 0 /dev/mapper/cachedev_0

e2fsck 1.44.1 (24-Mar-2018)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information

     2325930 inodes used (0.13%, out of 1799725056)
      187831 non-contiguous files (8.1%)
         857 non-contiguous directories (0.0%)
             # of inodes with ind/dind/tind blocks: 0/0/0
             Extent depth histogram: 2289445/27565/33
 12069212190 blocks used (83.83%, out of 14397792256)
           0 bad blocks
        4518 large files

     1944248 regular files
      372657 directories
           4 character device files
           1 block device file
           0 fifos
       57177 links
        9010 symbolic links (8872 fast symbolic links)
           1 socket
------------
     2383098 files

```

### After fsck reports all clean we can mount our volume!
```
mount /volume1
```

### Run some Synology specific checks and verify we're mounted:
```
synospace --map-file -d
synocheckshare
synocheckiscsitrg
ll /volume1/
grep volume1 /etc/samba/smb.share.conf
```


### Warning Indicator
At this point your filesystem should be healthy.  However DSM will still show a warning.  This is because the indicator that marks the volume healthy lives in a table that needs to be updated.  

Go back to DSM and for your volume check the file system, it should be able to unmount now.  If it can't it will tell you why (I had to disable docker via package manager).



---
via <https://github.com/jmiller0>
