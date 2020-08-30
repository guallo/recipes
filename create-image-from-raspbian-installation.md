# Create an image from a Raspbian installation

Also you may want to look at the recipe about [how to restore the image file](https://github.com/guallo/recipes/blob/master/restore-raspbian-image.md#restore-a-raspbian-image)
.

## Steps for Ubuntu 18.04

1. Move to a directory with enough space where to create the image file; replace `<path-to-directory-with-enough-space>` accordingly:

    ```bash
    cd <path-to-directory-with-enough-space>
    ```

2. Insert the medium that contains the Raspbian OS.

3. Locate the device and its partitions:

    ```bash
    lsblk
    
        NAME      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
        ...
        sdb         8:16   1  29,8G  0 disk 
        ├─sdb1      8:17   1   256M  0 part /media/<your-username>/boot
        └─sdb2      8:18   1  29,6G  0 part /media/<your-username>/rootfs
        ...
    ```
    
    In the output of the above command search for the two lines containing "*/media/\<your-username>/boot*" and "*/media/\<your-username>/rootfs*" as the "*MOUNTPOINT*"; they refer to the partitions of the device; we are interested in the column "*NAME*", where for this example:
    
    *  **sdb** is the `<device>`
    *  **sdb1** is the `<first-partition>`
    *  **sdb2** is the `<second-partition>`
    
    **Note that the above values may be different on your case.**

4. Umount the device partitions; replace `<first-partition>` and `<second-partition>` with the values from the previous step:

    ```bash
    sudo umount /dev/<first-partition>
    sudo umount /dev/<second-partition>
    ```

5. Make an image (*custom-raspbian.img*) of the device; replace `<device>` with the values from the **Step 3**:

    ```bash
    sudo cat /dev/<device> | dd of=custom-raspbian.img conv=fsync bs=$(( 1024 ** 2 * 4 ))
    ```

6. Eject the medium that contains the Raspbian OS, we do not need it anymore.

7. Setup loop devices for the created image:

    ```bash
    sudo losetup --find --partscan custom-raspbian.img
    ```

8. Locate the created loop devices:

    ```bash
    lsblk
    
        NAME      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
        ...
        loop6       7:6    0  29,8G  0 loop 
        ├─loop6p1 259:0    0   256M  0 loop 
        └─loop6p2 259:1    0  29,6G  0 loop 
        ...
    ```
    
    Search for the three lines containing the same "*SIZE*" values as the original ones in **Step 3**; they represent the *custom-raspbian.img* and its partitions; we are interested in the column "*NAME*", where for this example:
    
    *  **loop6** is the `<loop-device>` associated with the *custom-raspbian.img* file
    *  **loop6p1** is the `<first-loop-partition>`
    *  **loop6p2** is the `<second-loop-partition>`
    
    **Note that the above values may be different on your case.**

9. Open with *Gparted* the loop device associated with the *custom-raspbian.img*, replace `<loop-device>` with the values from previous step:

    ```bash
    sudo gparted /dev/<loop-device>
    ```

10. Shrink the last partition listed in *Gparted* to the minimum size possible **plus 1000 MB** approximately.

11. Close *Gparted*.

12. Detach the *custom-raspbian.img* file associated with the loop device, replace `<loop-device>` with the values from **Step 8**:

    ```bash
    sudo losetup --detach /dev/<loop-device>
    ```

13. Determine the sector size and the last sector of the second partition:

    ```bash
    fdisk -l custom-raspbian.img
    
        Disk custom-raspbian.img: 29,8 GiB, 32010928128 bytes, 62521344 sectors
        Units: sectors of 1 * 512 = 512 bytes
        Sector size (logical/physical): 512 bytes / 512 bytes
        I/O size (minimum/optimal): 512 bytes / 512 bytes
        Disklabel type: dos
        Disk identifier: 0xd9b3f436
    
        Device               Boot  Start      End  Sectors  Size Id Type
        custom-raspbian.img1        8192   532479   524288  256M  c W95 FAT32 (LBA)
        custom-raspbian.img2      532480 10725375 10192896  4,9G 83 Linux
    ```
    
    In this example the `<sector-size>` is **512** (taken from the line starting with "*Sector size*") and the `<last-sector>` is **10725375** (taken from the column "*End*" of the last listed device "*custom-raspbian.img2*").
    
    **Note that the above values may be different on your case.**

14. Truncate the *custom-raspbian.img* file to the last sector of the second partition; replace `<last-sector>` and `<sector-size>` with the values determined in the previous step:

    ```bash
    truncate --size=$(( (<last-sector> + 1) * <sector-size> )) custom-raspbian.img
    ```

15. Version the *custom-raspbian.img* file with the current date and time:

    ```bash
    mv custom-raspbian{,-"$(date '+%Y-%m-%d-%H-%M-%S')"}.img
    ```

16. Compress the versioned *custom-raspbian-\<version>.img* file into the *custom-raspbian-\<version>.img.gz* file; replace `<version>` accordingly:

    ```bash
    gzip custom-raspbian-<version>.img
    ```
