### *Production Incident: Surviving a 15,847-Request SSH Brute-Force Attack with Fail2Ban*

---

> *A real-world DevOps incident analysis — how a 3 AM alert revealed 15,847 failed SSH login attempts, and how Fail2Ban blocked 847 malicious IPs to restore server stability.*

---

### The Story Begins with a Phone Call

Last month at 3:20 AM, I received an alert from my server:

```
CRITICAL: CPU Usage 98% | Memory 94% | SSH Login Attempts: 15,847 in last 10 minutes
```

I was asleep. But I woke up immediately. CPU at 98%? Over 15,000 SSH login attempts? This wasn't normal.

I quickly SSH'd into the server. What I saw was terrifying:

```bash
$ sudo tail -f /var/log/auth.log
```

The screen was scrolling with thousands of login attempts:

```
Failed password for root from 185.220.101.1 port 54321 ssh2
Failed password for admin from 185.220.101.1 port 54322 ssh2
Failed password for root from 185.220.101.1 port 54323 ssh2
Failed password for ubuntu from 185.220.101.1 port 54324 ssh2
```

**One IP was making 50+ login attempts per second. This was a brute-force attack.**

---

### What Did I Do Wrong?

My server had SSH password authentication enabled. No IP blocking. No security measures.

I thought: "Who knows my server? Who would attack it?"

**Wrong assumption.**

Automated scanners run 24/7 on the internet. They scan every IP. Any IP with SSH open gets brute-forced immediately.

---

### What is Fail2Ban? In Simple Terms

Fail2Ban is a security tool that:
- Reads your log files
- Finds malicious IPs
- Automatically bans them
- Unbans after a specified time

**Example:** If an IP enters the wrong password 10 times in 5 minutes, Fail2Ban blocks that IP for 1 hour.

---

### Fail2Ban Installation and Setup

#### Step 1: Install

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install fail2ban -y

# CentOS/RHEL
sudo yum install fail2ban -y
```

#### Step 2: Copy Config File

```bash
# Keep original file
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit jail.local
sudo nano /etc/fail2ban/jail.local
```

#### Step 3: Configure SSH Jail

```ini
[DEFAULT]
# Default ban time: 1 hour
bantime = 3600

# 5 failures in 10 minutes triggers ban
findtime = 600
maxretry = 5

# Ban method
banaction = iptables-multiport

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 86400
```

#### Step 4: Start Fail2Ban

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo systemctl status fail2ban
```

---

### What Happened on My Server?

After enabling Fail2Ban, I saw:

```bash
$ sudo fail2ban-client status sshd
```

```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 23
|  |- Total failed:     15,847
`- Actions
   |- Currently banned: 847
   |- Total banned:     2,341
   `- Banned IP list:   185.220.101.1 185.220.101.2 45.155.205.233 ...
```

**847 IPs banned. 15,847 failed login attempts caught.**

---

### Custom Filters: Web Server Protection

Not just SSH, you can protect Apache, Nginx, WordPress with Fail2Ban.

#### WordPress Login Protection:

```bash
# Create filter file
sudo nano /etc/fail2ban/filter.d/wordpress.conf
```

```ini
[Definition]
failregex = ^<HOST> .* "POST /wp-login\.php
ignoreregex =
```

```bash
# Add to jail.local
[wordpress]
enabled = true
filter = wordpress
port = http,https
logpath = /var/log/apache2/access.log
maxretry = 5
bantime = 86400
```

---

### Real-time Monitoring

#### View current ban list:

```bash
# See all banned IPs
sudo iptables -L f2b-sshd -n

# Fail2Ban status
sudo fail2ban-client status
```

#### Manually unban an IP:

```bash
sudo fail2ban-client set sshd unbanip 185.220.101.1
```

#### Monitor log file:

```bash
# See bans in real-time
sudo tail -f /var/log/fail2ban.log
```

---

### Advanced Config: Multi-Server Setup

If you have multiple servers, you can share ban lists using Redis:

```ini
[DEFAULT]
banaction = iptables-multiport

# Use Redis
backend = redis
redis_host = 127.0.0.1
redis_port = 6379
redis_key = fail2ban:%(name)s
```

```bash
# Redis install
sudo apt install redis-server

# Fail2Ban Redis config
sudo nano /etc/fail2ban/jail.local
```

---

### Lessons I Learned

1. **"Who knows my server?" — Wrong question.** Everyone knows it.

2. **Disable password authentication.** Use SSH keys.

3. **Fail2Ban must be running.** It's your first line of defense.

4. **Never skip monitoring.** Set up alerts.

5. **Update regularly.** Apply new security patches.

---

### Final Words

That night taught me: **Security is not optional, it's essential.**

Fail2Ban is a simple tool. But it can protect your server from thousands of attacks.

**Install Fail2Ban today. Keep your server secure.**

---

**Tags:** #Fail2Ban #ServerSecurity #Linux #CyberSecurity #SSH #BruteForceAttack #DevOps #SystemAdministration #OpenSource #IncidentResponse

---

## 📂 GitHub Repository

📋 **Explore the complete article and troubleshooting resources:**

👉 **[GitHub Repository: Medium-Blog](https://github.com/iqbalbd37/Medium-Blog)**

📄 **Read the full Fail2Ban Incident Report in Markdown:**

👉 [Production Incident: Surviving a 15,847-Request SSH Brute-Force Attack — Full Article](https://github.com/iqbalbd37/Medium-Blog/blob/main/articles/12-fail2ban-incident/fail2ban-incident.md)

### What You'll Find in This Repository

- ✅ Real-world SSH brute-force attack analysis
- ✅ Fail2Ban setup and emergency response walkthrough
- ✅ Custom filters for WordPress and web server protection
- ✅ Multi-server ban list sharing with Redis
- ✅ DevOps incident documentation and lessons learned

⭐ **If you find these resources useful, consider starring the repository on GitHub!**

*Learn, troubleshoot, and share your DevOps knowledge.*

---
