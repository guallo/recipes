# Restore a Raspbian image

Also you may want to look at the recipe about [how to make the image file](https://github.com/guallo/recipes/blob/master/create-image-from-raspbian-installation.md#create-an-image-from-a-raspbian-installation)
.

## Steps for Ubuntu 18.04

1. Move to the directory containing the image file; replace `<path-to-directory-containing-image-file>` accordingly:

    ```bash
    cd <path-to-directory-containing-image-file>
    ```

2. Insert the medium to write the image in.

3. Locate the device and its partitions (if any):

    ```bash
    lsblk
    
        NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
        ...
        sdb      8:16   1  28,9G  0 disk 
        ├─sdb1   8:17   1   256M  0 part /media/<your-username>/<first-partition-label>
        └─sdb2   8:18   1   6,1G  0 part /media/<your-username>/<second-partition-label>
        ...
    ```
    
    In this example, **sdb** is the `<device>` while **sdb1** and **sdb2** are its partitions; 
    let's call them `<first-partition>` and `<second-partition>` respectively.
    
    If your device have some partitions that are already mounted then you can search for them 
    in the output of the above command by looking at the "*MOUNTPOINT*" column.
    
    **Note that the above values may be different in your case. For example your 
    device may be named another way and have more partitions or no one at all.**

4. Umount the device partitions (if any); replace `<first-partition>`, `<second-partition>` ... `<nth-partition>` with the values from the previous step:

    ```bash
    sudo umount /dev/<first-partition>
    sudo umount /dev/<second-partition>
    ...
    sudo umount /dev/<nth-partition>
    ```

5. Write the image to the device; replace `<custom-raspbian-version.img.gz>` with your gzip'ed image file 
   and replace `<device>` with the values from the **Step 3**:

    ```bash
    cat <custom-raspbian-version.img.gz> | gunzip | sudo dd of=/dev/<device> conv=fsync bs=$(( 1024 ** 2 * 4 ))
    ```

6. Open the device with *Gparted*; replace `<device>` with the values from the **Step 3**:

    ```bash
    sudo gparted /dev/<device>
    ```

7. Resize the last partition listed in *Gparted* to the maximum size possible.

8. Close *Gparted*.
