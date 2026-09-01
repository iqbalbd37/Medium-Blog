# Redis on RHEL 9: Complete Production Setup Guide

![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![RHEL](https://img.shields.io/badge/RHEL-EE0000?style=for-the-badge&logo=redhat&logoColor=white)
![DevOps](https://img.shields.io/badge/DevOps-FF6B35?style=for-the-badge&logo=devops&logoColor=white)

## Overview

A complete step-by-step guide to installing, securing, tuning, monitoring, and backing up Redis on a single RHEL 9 server for production workloads.

**Read on Medium:** [Click Here](https://medium.com/@iqbalbd37/redis-on-rhel-9-complete-production-setup-guide-for-session-cache-server-on-a-single-host-d267685d6893)

---

## What You'll Learn

- Installing Redis on RHEL 9
- Securing the server with authentication
- Performance tuning and optimization
- SELinux and Firewall configuration
- Persistence configuration
- Monitoring and backup automation
- Connecting Docker containers

---

## Table of Contents

1. [Introduction](#introduction)
2. [Architecture](#architecture)
3. [Step 1 — Prepare RHEL 9](#step-1--prepare-rhel-9)
4. [Step 2 — Install Redis](#step-2--install-redis)
5. [Step 3 — Configure Redis](#step-3--configure-redis)
6. [Step 4 — Kernel Performance Tuning](#step-4--kernel-performance-tuning)
7. [Step 5 — Disable Transparent Huge Pages](#step-5--disable-transparent-huge-pages)
8. [Step 6 — Configure Firewall](#step-6--configure-firewall)
9. [Step 7 — Configure SELinux](#step-7--configure-selinux)
10. [Step 8 — Create Required Directories](#step-8--create-required-directories)
11. [Step 9 — Start Redis](#step-9--start-redis)
12. [Step 10 — Verify Redis](#step-10--verify-redis)
13. [Step 11 — Monitoring](#step-11--monitoring)
14. [Step 12 — Connect Docker Applications](#step-12--connect-docker-applications)
15. [Step 13 — Automated Backup](#step-13--automated-backup)
16. [Step 14 — Production Verification](#step-14--production-verification)
17. [Step 15 — Recovery Test](#step-15--recovery-test)

---

## Introduction

Redis is one of the fastest in-memory data stores available today. It is commonly used for:

- **Session Storage**
- **Application Cache**
- **Rate Limiting**
- **Message Queue**
- **Leaderboards**
- **Pub/Sub**
- **Distributed Locks**

This guide walks through building a **production-ready Redis server on a single RHEL 9 host**.

---

## Step 1 — Prepare RHEL 9

```bash
sudo dnf update -y && sudo dnf upgrade -y
```

Install useful administration tools:

```bash
sudo dnf install wget curl vim net-tools policycoreutils-python-utils -y
```

---

## Step 2 — Install Redis

Enable the EPEL repository:

```bash
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm -y
```

Install Redis:

```bash
sudo dnf install redis -y
```

Verify installation:

```bash
redis-server --version
redis-cli --version
```

---

## Step 3 — Configure Redis

Backup the original configuration:

```bash
sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.original.backup
```

Edit Redis configuration:

```bash
sudo vim /etc/redis/redis.conf
```

### Network Settings

```
bind 0.0.0.0
port 6379
protected-mode yes
tcp-backlog 511
timeout 3600
tcp-keepalive 300
```

### Authentication

```bash
requirepass YourVeryStrong_P@ssw0rd_2024_ChangeIt
```

### Memory Settings

```
maxmemory 512mb
maxmemory-policy allkeys-lru
maxmemory-samples 5
```

### Persistence Settings

```
save 900 1
save 300 10
save 60 10000

stop-writes-on-bgsave-error no
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /var/lib/redis
```

---

## Step 4 — Kernel Performance Tuning

```bash
sudo vim /etc/sysctl.d/99-redis-tuning.conf
```

Add:

```
net.core.somaxconn = 65535
vm.overcommit_memory = 1
net.ipv4.tcp_max_syn_backlog = 4096
net.core.netdev_max_backlog = 10000
vm.swappiness = 1
```

Apply:

```bash
sudo sysctl -p /etc/sysctl.d/99-redis-tuning.conf
```

---

## Step 5 — Disable Transparent Huge Pages

```bash
sudo vim /etc/systemd/system/disable-thp.service
```

Add:

```ini
[Unit]
Description=Disable Transparent Huge Pages
After=sysinit.target local-fs.target
Before=redis.service

[Service]
Type=oneshot
ExecStart=/bin/sh -c "echo never > /sys/kernel/mm/transparent_hugepage/enabled"

[Install]
WantedBy=basic.target
```

Enable it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable disable-thp.service
sudo systemctl start disable-thp.service
```

---

## Step 6 — Configure Firewall

```bash
sudo firewall-cmd --permanent --add-port=6379/tcp
sudo firewall-cmd --reload
```

---

## Step 7 — Configure SELinux

```bash
sudo semanage fcontext -a -t redis_port_t "/etc/redis(/.*)?"
sudo semanage fcontext -a -t redis_var_lib_t "/var/lib/redis(/.*)?"
sudo semanage fcontext -a -t redis_log_t "/var/log/redis(/.*)?"
sudo restorecon -Rv /etc/redis
sudo restorecon -Rv /var/lib/redis
sudo restorecon -Rv /var/log/redis
```

Enable Redis booleans:

```bash
sudo setsebool -P redis_enable_notify 1
sudo setsebool -P redis_use_nfs 1
```

Register Redis port:

```bash
sudo semanage port -a -t redis_port_t -p tcp 6379
```

---

## Step 8 — Create Required Directories

```bash
sudo mkdir -p /var/lib/redis
sudo mkdir -p /var/log/redis
sudo mkdir -p /var/run/redis

sudo chown -R redis:redis /var/lib/redis
sudo chown -R redis:redis /var/log/redis
sudo chown -R redis:redis /var/run/redis

sudo chmod 750 /var/lib/redis
sudo chmod 750 /var/log/redis
```

---

## Step 9 — Start Redis

```bash
sudo systemctl start redis && sudo systemctl enable redis
```

---

## Step 10 — Verify Redis

Ping Redis:

```bash
redis-cli -a "YourVeryStrong_P@ssw0rd_2024_ChangeIt" ping
```

**Expected output:** `PONG`

### Session Test

```bash
SET session:user123 '{"user_id":1,"name":"Test User","role":"admin"}' EX 3600
GET session:user123
TTL session:user123
```

### Cache Test

```bash
SET cache:product:1001 "Product Data Here" EX 300
GET cache:product:1001
```

---

## Step 11 — Monitoring

Watch logs:

```bash
sudo tail -f /var/log/redis/redis.log
```

Live statistics:

```bash
redis-cli -a "password" --stat
```

---

## Step 12 — Connect Docker Applications

```bash
docker run -d \
  --name your-container \
  -p 8100:8100 \
  -e REDIS_HOST=<server-ip> \
  -e REDIS_PORT=6379 \
  -e REDIS_PASSWORD=YourPassword \
  -e REDIS_DB=0 \
  -e SESSION_TTL=3600 \
  -e CACHE_TTL=300 \
  --restart unless-stopped \
  your-image:latest
```

---

## Step 13 — Automated Backup

Create backup script:

```bash
sudo vim /usr/local/bin/redis-backup.sh
```

Add:

```bash
#!/bin/bash
BACKUP_DIR="/backup/redis"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p $BACKUP_DIR
redis-cli -a "YourPassword" SAVE
cp /var/lib/redis/dump.rdb $BACKUP_DIR/dump_$DATE.rdb
find $BACKUP_DIR -name "*.rdb" -mtime +7 -delete
echo "Redis backup completed at $DATE" >> /var/log/redis-backup.log
```

Make executable:

```bash
sudo chmod +x /usr/local/bin/redis-backup.sh
```

Create cron jobs:

```bash
0 2 * * * /usr/local/bin/redis-backup.sh
0 */6 * * * redis-cli -a "YourPassword" BGREWRITEAOF
```

---

## Step 14 — Production Verification

```bash
echo "=== Redis Version ==="
redis-server --version

echo "=== Service Status ==="
sudo systemctl status redis --no-pager

echo "=== Redis Ping Test ==="
redis-cli -a "YourPassword" ping

echo "=== Memory Info ==="
redis-cli -a "YourPassword" INFO memory | head -10

echo "=== Firewall ==="
sudo firewall-cmd --list-ports

echo "=== SELinux ==="
sudo getenforce

echo "=== Redis is Ready for Production ==="
```

[![Medium](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://medium.com/@iqbalbd37)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/iqbalbd37)