# How to Install Apache Tomcat on RHEL 9 with Java 21

![Tomcat](https://img.shields.io/badge/Apache%20Tomcat-F8DC75?style=for-the-badge&logo=apache-tomcat&logoColor=black)
![RHEL](https://img.shields.io/badge/RHEL-EE0000?style=for-the-badge&logo=redhat&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)

## Overview

Apache Tomcat is one of the most popular Java application servers for deploying Java web applications. In this guide, you'll learn how to install Apache Tomcat on Red Hat Enterprise Linux (RHEL) 9 using OpenJDK 21 and configure it to run as a systemd service.

**Read on Medium:** [Click Here](https://medium.com/@iqbalbd37/how-to-install-apache-tomcat-on-rhel-9-with-java-21-0a18e1520e56)

---

## Prerequisites

- A RHEL 9 server
- A user with `sudo` privileges
- Internet connectivity to download packages and Tomcat
- Firewall access to port `8080`

---

## Table of Contents

1. [Step 1: Update the System](#step-1-update-the-system)
2. [Step 2: Install Java 21](#step-2-install-java-21)
3. [Step 3: Create a Dedicated Tomcat User](#step-3-create-a-dedicated-tomcat-user)
4. [Step 4: Download Apache Tomcat](#step-4-download-apache-tomcat)
5. [Step 5: Configure File Permissions](#step-5-configure-file-permissions)
6. [Step 6: Create a systemd Service](#step-6-create-a-systemd-service)
7. [Step 7: Enable and Start Tomcat](#step-7-enable-and-start-tomcat)
8. [Step 8: Configure the Firewall](#step-8-configure-the-firewall)
9. [Step 9: Configure SELinux](#step-9-configure-selinux)
10. [Step 10: Access Tomcat](#step-10-access-tomcat)
11. [Useful Commands](#useful-tomcat-commands)

---

## Step 1: Update the System

```bash
sudo dnf update -y
```

---

## Step 2: Install Java 21

```bash
sudo dnf install -y java-21-openjdk java-21-openjdk-devel
```

Verify the installation:

```bash
java -version
```

---

## Step 3: Create a Dedicated Tomcat User

```bash
sudo useradd -r -m -U -d /opt/tomcat -s /bin/false tomcat
```

---

## Step 4: Download Apache Tomcat

```bash
cd /tmp
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.57/bin/apache-tomcat-10.1.57.tar.gz
```

Extract the archive:

```bash
sudo mkdir -p /opt/tomcat
sudo tar -xzf apache-tomcat-10.1.57.tar.gz \
  -C /opt/tomcat \
  --strip-components=1
```

---

## Step 5: Configure File Permissions

```bash
sudo chown -R tomcat:tomcat /opt/tomcat
sudo chmod +x /opt/tomcat/bin/*.sh
```

---

## Step 6: Create a systemd Service

```bash
sudo vim /etc/systemd/system/tomcat.service
```

Add the following configuration:

```ini
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking
User=tomcat
Group=tomcat
Environment="JAVA_HOME=/usr/lib/jvm/java-21-openjdk"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_BASE=/opt/tomcat"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M"
Environment="JAVA_OPTS=-Djava.security.egd=file:/dev/urandom"
ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh
SuccessExitStatus=143
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

---

## Step 7: Enable and Start Tomcat

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now tomcat
```

Verify that the service is running:

```bash
sudo systemctl status tomcat
```

---

## Step 8: Configure the Firewall

Allow incoming traffic on port **8080**:

```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

Verify:

```bash
sudo firewall-cmd --list-ports
```

---

## Step 9: Configure SELinux

Install the required utilities:

```bash
sudo dnf install -y policycoreutils-python-utils
```

Add the SELinux rule:

```bash
sudo semanage port -a -t http_port_t -p tcp 8080
```

If the port already exists, modify it:

```bash
sudo semanage port -m -t http_port_t -p tcp 8080
```

---

## Step 10: Access Tomcat

Open your browser and navigate to:

```
http://YOUR_SERVER_IP:8080
```

Or verify locally:

```bash
curl http://localhost:8080
```

---

## Useful Tomcat Commands

| Command | Description |
|---------|-------------|
| `sudo systemctl start tomcat` | Start Tomcat |
| `sudo systemctl stop tomcat` | Stop Tomcat |
| `sudo systemctl restart tomcat` | Restart Tomcat |
| `sudo systemctl status tomcat` | Check service status |
| `sudo journalctl -u tomcat -f` | View service logs |
| `tail -f /opt/tomcat/logs/catalina.out` | View Tomcat logs |

---

## Tags

`#Tomcat` `#RHEL9` `#Java` `#DevOps` `#Linux` `#WebServer` `#JavaApps` `#Tutorial`

---

[![Medium](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://medium.com/@iqbalbd37)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/iqbalbd37)