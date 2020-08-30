# Setup SFTP server (tested on Ubuntu Server 20.04)

1. Create the system group `sftponly`:
    ```bash
    sudo addgroup --system sftponly
    ```

2. Create the root directory `/sftponly` where to store/retrieve the files to/from:
    ```bash
    sudo mkdir /sftponly
    ```

3. Only if you have a dedicated clean `ext4` filesystem mounted on directory `/sftponly` (**recommended**, also you might want to see how to [Setup a Logical Volume](https://github.com/guallo/recipes/-/blob/master/setup-logical-volume.md#setup-a-logical-volume-using-lvm2-tested-on-ubuntu-server-2004)):
    ```bash
    sudo chown root:root /sftponly
    sudo chmod 755 /sftponly
    ```

4. Create a system user `<USERNAME>` with its own password and store subdirectory. 
Repeat this step for every user that is intended to use the service. Replace `<USERNAME>` accordingly:
    ```bash
    sudo adduser --system --ingroup sftponly <USERNAME>
    sudo passwd <USERNAME>
    
    sudo mkdir /sftponly/<USERNAME>
    sudo chown <USERNAME>:sftponly /sftponly/<USERNAME>
    sudo chmod 700 /sftponly/<USERNAME>  # or use 750 if you want to give 'access files' permissions to sftponly-users
    ```

5. Install, configure and restart the SSH server (which is the provider of the SFTP service):
    ```bash
    sudo apt-get install ssh
    sudo tee -a /etc/ssh/sshd_config > /dev/null << 'EOF'
    
    Match Group sftponly
        ForceCommand internal-sftp
        ChrootDirectory /sftponly
        PasswordAuthentication yes
    
    EOF
    sudo systemctl restart ssh
    ```
