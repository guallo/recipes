# Setup a Logical Volume using LVM2 (tested on Ubuntu Server 20.04)

1. Supposing you have some disks you want to use for creating a *Logical Volume*. Let's say:
    - `/dev/sdX` the disk X,
    - `/dev/sdY` the disk Y and,
    - `/dev/sdZ` the disk Z.
2. **Be aware of saving the data of those disks (if necessary) before continuing with following steps.**
3. Wipe those disks:
    ```bash
    sudo wipefs --all --force /dev/sdX /dev/sdY /dev/sdZ
    ```
4. Install the `lvm2` package:
    ```bash
    sudo apt-get install lvm2
    ```
5. Create a volume group named `vg1` (you can choose a different name) around those disks:
    ```bash
    sudo vgcreate vg1 /dev/sdX /dev/sdY /dev/sdZ
    ```
6. Create the *Logical Volume* with name `lv1` (you can choose a different name) and all the unallocated space of the volume group `vg1`:
    ```bash
    sudo lvcreate --name lv1 -l 100%FREE vg1
    ```
7. Create an `ext4` filesystem in the *Logical Volume* `/dev/mapper/vg1-lv1` (yours could have a different path depending on the names chosen on previous steps):
    ```bash
    sudo mkfs.ext4 /dev/mapper/vg1-lv1
    ```
8. Add an `fstab entry` to automatically mount the *Logical Volume*'s filesystem at boot time. Replace `<MOUNT-POINT>` with a directory path 
(e.g. `/sftponly` in case you come from the [Setup SFTP server](https://github.com/guallo/recipes/blob/master/setup-sftp-server.md#setup-sftp-server-tested-on-ubuntu-server-2004) recipe):
    ```bash
    sudo tee -a /etc/fstab > /dev/null <<< '/dev/mapper/vg1-lv1 <MOUNT-POINT> ext4 defaults 0 0'
    ```
9. Mount the *Logical Volume*'s filesystem:
    ```bash
    sudo mount /dev/mapper/vg1-lv1
    ```
