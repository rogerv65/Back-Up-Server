# Back-Up-Server
## Introduction
The purpose of this lab was to gain exposure to configuration of a file share, and create a back-up for important documents. There were multiple paths available (TrueNAS), but after research I opted down the path of **Ubuntu Server** paired with the functionality of **Samba**. **Samba** provides various tools for configuring **Ubuntu Server** to share network resources with Windows clients. **Samba** has multiple functions, but the main function I am planning to utilize is the file sharing services. I want to map a network drive to a file share, so that it acts as a back-up server for important documents. It’s also going to be set-up so that only one user (myself) can access it with the appropriate credentials.
## Set-up & Configuration
1. Begin by downloading the .iso file of Ubuntu Server. Download .iso here: https://ubuntu.com/download/server
2. Within the virtual environment, configure a VM w/ the .iso and with the following specs: 2 CPUs, 3 GB, and 20GB disk (these specs are based on my needs for this project). 
3. Go through the Ubuntu Server installer and complete the installation. *Note, Ubuntu Server doesn’t have a GUI, so everything is CLI*
4. Next is installing Samba. Start by installing the samba package. Enter the following command into the server terminal:

    `sudo apt install samba`
   
6. Navigate to the main Samba configuration file located in */etc/samba/smb.conf*. Do so with the `cd` command. In order to access **smb.conf**, ensure to run `sudo nano smb.conf`.
7. Once in editor, edit the **workgroup** parameter in the *[global]* section & change it to match the environment.

    workgroup = *your example*
   
8. At the bottom of the file, create a new section for the directory you want to share. In my case the following will be set up:

    *[share name]*

         comment = *Your comment here*
   
         path = /srv/samba/ *share name* # path to directory that is being shared
   
         browsable = yes  # Enables Windows clients to browse shared directory via file explorer.
   
         guest ok = yes  # Clients can connect w.o a shared password
   
         read only = no # Allows for write permissions
   
         create mask = 0755 # Permissions that new files will have when created
   
9. Exit and save these configurations into **smb.conf**.
10. Next, we need to create the directory and change permissions. Run the following in a terminal:
    
    `sudo mkdir -p /srv/samba/*share name*`
    
    `sudo chown nobody:nogroup /srv/samba/*share name*/`
    
11. Lastly we need to restart the Samba services for changes to take place. Restart service with the following:

    `sudo systemctl restart smbd.service nmbd.service`

    *Can also reboot the computer as well.*
    
12. Now to map the file share to a network drive on a Windows client, open the terminal and enter the following command:

    `net use *letter of drive*: \\*server name*\*share name*`
    
14. With this, the file share can be found as a network drive within file explorer under This PC. The share is accessible with anyone within the local network.

## Share Access Controls
15. We will now begin controlling access to the file share by creating an individual admin user. Samba uses the existing Linux user to create a Samba user. To create our first Samba user enter the following:

    `sudo smbpasswd -a *Linux user name*`
    
    *Will prompt you for a password, the password does not have to match the password for your Linux user.*
    *If the user does not exist, the command will fail.*
    
16. Next, we will define explicit read and write permissions for the newly created user to the share. Edit the **smb.conf** file found in */etc/samba* with the following entries your *[share]* entry:

    read list = *samba user name*
    
    write list = *samba user name*
    
17. Alternatively, you could give administrative permissions to a user by entering the following:

    admin users = *samba user name*
    
18. Save the changes and run the following for the configurations to take effect:

    `sudo smbcontrol smbd reload-config`
19. Next, we will begin modification of filesystem permissions. Traditional Linux file permissions do not map well to Windows NT ACLs. To counteract this, edit  */etc/fstab* and add the acl option:

    UUID=66bcdd2e-8861-4fb0-b7e4-e61c569fe17d /srv  ext3    noatime,relatime,acl 0       1
    
20. Then remount with:

    `sudo mount -v -o remount /srv`
    
21. Lastly, to match the Samba configuration, we will enter the following:

    `sudo chown -R *samba user* /srv/samba/share/`

With this, attempting to map the file share to a network drive will require credentials. Simply enter the credentials of the newly created Samba user!

