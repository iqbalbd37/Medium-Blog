# How to Install and Configure an FTP Server on Ubuntu 22.04

![FTP](https://img.shields.io/badge/FTP-00599C?style=for-the-badge&logo=filezilla&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![VSFTPD](https://img.shields.io/badge/VSFTPD-2ECC71?style=for-the-badge&logo=server&logoColor=white)

## Overview

A complete guide to installing and configuring an FTP server on Ubuntu 22.04 using VSFTPD and connecting to it from a Windows machine.

**Read on Medium:** [Click Here](https://medium.com/@iqbalbd37/how-to-install-and-configure-an-ftp-server-on-ubuntu-22-04-and-connect-from-windows-eda97143112c)

---

## Environment

| Component | Details |
|-----------|---------|
| Ubuntu FTP Server | 192.168.0.12 |
| Windows Client | 192.168.0.10 |
| FTP Software | VSFTPD (Very Secure FTP Daemon) |

---

## Table of Contents

1. [Step 1: Update Ubuntu](#step-1-update-ubuntu)
2. [Step 2: Install VSFTPD](#step-2-install-vsftpd)
3. [Step 3: Backup Configuration](#step-3-backup-the-default-configuration)
4. [Step 4: Create FTP User](#step-4-create-an-ftp-user)
5. [Step 5: Create FTP Directories](#step-5-create-ftp-directories)
6. [Step 6: Configure VSFTPD](#step-6-configure-vsftpd)
7. [Step 7: Restart Service](#step-7-restart-the-ftp-service)
8. [Step 8: Configure Firewall](#step-8-configure-the-firewall)
9. [Step 9: Verify Listening](#step-9-verify-ftp-is-listening)
10. [Step 10: Test Locally](#step-10-test-ftp-locally)
11. [Common Errors](#common-errors)
12. [Connecting from Windows](#connecting-from-windows)

---

## Step 1: Update Ubuntu

```bash
sudo apt update -y
sudo apt upgrade -y
```

---

## Step 2: Install VSFTPD

```bash
sudo apt install vsftpd -y
```

Verify service:

```bash
sudo systemctl status vsftpd
```

Enable auto-start:

```bash
sudo systemctl enable vsftpd
```

---

## Step 3: Backup the Default Configuration

```bash
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
```

---

## Step 4: Create an FTP User

```bash
sudo adduser ftpuser
```

Set a strong password when prompted.

---

## Step 5: Create FTP Directories

```bash
sudo mkdir -p /home/ftpuser/ftp/upload
```

Configure ownership and permissions:

```bash
sudo chown nobody:nogroup /home/ftpuser/ftp
sudo chmod a-w /home/ftpuser/ftp
sudo chown ftpuser:ftpuser /home/ftpuser/ftp/upload
```

---

## Step 6: Configure VSFTPD

```bash
sudo nano /etc/vsftpd.conf
```

Add or modify:

```
listen=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
chroot_local_user=YES
allow_writeable_chroot=YES
user_sub_token=$USER
local_root=/home/$USER/ftp
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100
userlist_enable=YES
```

---

## Step 7: Restart the FTP Service

```bash
sudo systemctl restart vsftpd
```

Check status:

```bash
sudo systemctl status vsftpd
```

---

## Step 8: Configure the Firewall

If UFW is enabled:

```bash
sudo ufw allow 20/tcp
sudo ufw allow 21/tcp
sudo ufw allow 40000:40100/tcp
sudo ufw reload
```

Check status:

```bash
sudo ufw status
```

---

## Step 9: Verify FTP is Listening

```bash
sudo ss -tulpn | grep :21
```

---

## Step 10: Test FTP Locally

Install FTP client:

```bash
sudo apt install ftp -y
```

Connect locally:

```bash
ftp localhost
```

Login with:
- **Username:** ftpuser
- **Password:** ********

Test commands:

```
ls
pwd
cd upload
```

---

## Common Errors

### Error: "500 OOPS: cannot read user list file"

**Cause:** VSFTPD is configured to use a user list file, but the file does not exist.

**Solution 1:** Create the user list file:

```bash
sudo touch /etc/vsftpd.user_list
sudo chmod 644 /etc/vsftpd.user_list
echo "ftpuser" | sudo tee -a /etc/vsftpd.user_list
sudo systemctl restart vsftpd
```

**Solution 2:** Disable user list checking:

```bash
sudo nano /etc/vsftpd.conf
```

Change `userlist_enable=YES` to `userlist_enable=NO`

---

### Error: Service Fails with status=2

**Cause:** Configuration issue, often having both `listen=YES` and `listen_ipv6=YES` enabled.

**Solution:** Use only one:

```
listen=YES
#listen_ipv6=YES
```

---

## Connecting from Windows

### Method 1: File Explorer

Open File Explorer and enter:

```
ftp://192.168.0.12
```

Enter your FTP username and password when prompted.

---

### Method 2: Command Prompt

```cmd
ftp 192.168.0.12
```

Commands:

```
dir
put test.txt
get sample.txt
bye
```

---

### Method 3: FileZilla (Recommended)

Configure:

| Field | Value |
|-------|-------|
| Host | 192.168.0.12 |
| Port | 21 |
| Username | ftpuser |
| Password | YourPassword |

Click **Quick Connect**.

---

## Connectivity Testing

From Windows:

```powershell
ping 192.168.0.12
Test-NetConnection 192.168.0.12 -Port 21
```

Expected output:

```
TcpTestSucceeded : True
```

[![Medium](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://medium.com/@iqbalbd37)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/iqbalbd37)