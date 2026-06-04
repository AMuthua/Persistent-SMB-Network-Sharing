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