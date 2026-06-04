# Persistent SMB Network Sharing
A beginner-friendly technical guide to mounting a remote Windows network share onto a headless Ubuntu Server using SMB/CIFS. Covers hidden credential security, folder permissions, and automated persistence via /etc/fstab to handle system reboots or power outages.




Welcome! This repository is a practical, step-by-step guide designed to teach you how to link a remote Windows network storage share directly into a headless Ubuntu Server's file system. 

Instead of copying files manually or losing your connection every time your server restarts, we will build a secure, persistent, and self-healing network tunnel using native Linux utilities.



---

## The Core Concept: What Are We Solving?

Imagine you have a central headless Linux server, and you want it to store backups or access files located on another machine specifically Windows across the room. 

If you just connect to it temporarily, the connection breaks the second there is a power cut or a system reboot. To fix this, we use two massive Linux building blocks:
1. **Samba/CIFS:** The communication protocol that allows Linux and Windows systems to talk and share folders over a local network.
2. **`/etc/fstab` (File System Table):** A powerful configuration registry inside the Linux kernel that mounts storage devices automatically the moment the operating system boots up.

## Step-by-Step Deployment Guide

### Step 1: Install the Necessary Network Utilities
By default, a clean Ubuntu Server doesn't know how to parse Windows filesystems. We need to install the core CIFS tools first. 

Connect to your server via SSH and run:

```bash

sudo apt update
sudo apt install cifs-utils -y

```

### Step 2: Create a Local Injection Point (The Mount Folder)
We need a standard directory inside our Linux filesystem where the remote network files will physically appear. We will create this inside the `/mnt` directory:

```bash

sudo mkdir -p /mnt/network-storage

```

### Step 3: Hide and Secure Your Network Credentials

Putting a plain-text Windows username and password directly into a public configuration file is a massive security risk. Instead, we isolate them inside a hidden file restricted by strict Linux file system permissions.

**Personal Advice:** _Best practice would be to hide it into the root user, as it's common practice to not use that user._

1. Create a hidden credentials file in your home directory:
```Bash
   touch ~/.smbcredentials
```

3. Lock down the file permissions so **only your user** can read or write to it:
```Bash
   chmod 600 ~/.smbcredentials
```
    
4. Open the file with a text editor (`nano ~/.smbcredentials`) and add your remote machine's details:
 

```Bash
   username=your_network_username
   password=your_network_password
   domain=WORKGROUP
```

_Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`)._

### Step 4: Automate the Mounting Process (`/etc/fstab`)

Now, we tell the Linux kernel to automatically establish the network tunnel every time the machine boots up. We do this by editing the File System Table:

Bash

```Bash
sudo nano /etc/fstab
```

Scroll to the absolute bottom of the file and append this single configuration line:

Plaintext

```Bash
//192.168.1.100/Shared-Backups  /mnt/network-storage  cifs  credentials=/root/.smbcredentials,iocharset=utf8,uid=1000,gid=1000  0  0
```

#### Breaking Down the Configuration Parameters:

- **`//192.168.1.100/Shared-Backups`**: The local network path to the shared folder on the remote machine.

- **`/mnt/network-storage`**: The local folder we created in Step 2 where the files will appear.

- **`cifs`**: Tells the kernel to use the Samba network file-sharing protocol.

- **`credentials=...`**: Points directly to our hidden, locked-down password file.

- **`uid=1000,gid=1000`**: Tells Linux that your local user account owns these files, preventing annoying "Permission Denied" errors when reading or writing data.

### Step 5: Test and Execute the Live Pipeline

You don't need to reboot your server to check if your configuration works. You can force the operating system to parse the `/etc/fstab` file immediately by running:

Bash

```Bash
sudo mount -a
```

_If the command runs silently without throwing errors, your tunnel is live!_

### Verification: Testing Your Success

To prove that the files are streaming over the network successfully, run a list directory command on your local folder:

Bash

```Bash
ls -l /mnt/network-storage
```

You will instantly see all the files and folders hosted on the remote computer sitting right inside your Linux terminal. If the server undergoes an unannounced power failure or a manual reboot, it will execute this handshake entirely on its own the moment it wakes up!


#### Note: 
1. Whenever you change your Windows Login credentials. You **MUST** update the `./smbcredentials` in the **root** user where the file is located. 
   

~By Alexander Muthua. 