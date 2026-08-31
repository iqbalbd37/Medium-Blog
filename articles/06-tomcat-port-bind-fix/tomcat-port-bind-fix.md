# Incident Report: Tomcat Port 80/443 Bind Failure

## Metadata

| Field | Value |
|-------|-------|
| **Incident ID** | INC-2026-0830-001 |
| **Date** | 30 August 2026 |
| **Severity** | P2 — Service Degraded |
| **Duration** | ~15 minutes |
| **Status** | Resolved |
| **Server** | `app-server-01.prod.internal` |
| **Service** | Apache Tomcat 10.1.15 |
| **Run-as User** | `appuser` (non-root) |
| **JDK** | Oracle JDK 21.0.2 |

---

## Executive Summary

Tomcat failed to bind to ports 80 and 443 due to privileged port restrictions for non-root users. Applying `setcap cap_net_bind_service` introduced a secondary issue: AT_SECURE mode prevented dynamic library resolution. Both issues were resolved through capabilities and ldconfig configuration.

---

## Timeline

| Time (UTC) | Event |
|------------|-------|
| 10:47 | Tomcat service restarted |
| 10:47 | Ports 80/443 not bound — only shutdown port 8005 listening |
| 10:48 | Root cause identified: non-root user lacks CAP_NET_BIND_SERVICE |
| 10:49 | `setcap cap_net_bind_service=+ep` applied to java binary |
| 10:49 | Secondary issue: libjli.so not found (AT_SECURE mode) |
| 10:50 | JDK lib paths registered with ldconfig |
| 10:51 | Tomcat restarted successfully |
| 10:55 | Ports 80/443 confirmed listening via curl verification |

---

## Root Cause Analysis

### Primary Cause: Privileged Port Restriction

Linux restricts binding to ports below 1024 to root (UID 0) or processes with `CAP_NET_BIND_SERVICE`. Tomcat was running as `appuser`, which had neither.

```
Tomcat process: appuser (UID 1001)
Required:       root (UID 0) OR CAP_NET_BIND_SERVICE
Result:         java.net.BindException: Permission denied
```

### Secondary Cause: AT_SECURE Library Resolution

Applying `setcap` to the Java binary caused the kernel to run it in AT_SECURE mode for non-root users. This disabled `LD_LIBRARY_PATH` and restricted library search to system paths.

```
Binary: /usr/lib/jvm/jdk-21-oracle-x64/bin/java
Capability: cap_net_bind_service=+ep
AT_SECURE: Activated for non-root users
Effect: libjli.so not found in restricted search paths
```

---

## Resolution

### Step 1: Grant Bind Capability

```bash
setcap 'cap_net_bind_service=+ep' /usr/lib/jvm/jdk-21-oracle-x64/bin/java
```

### Step 2: Register JDK Libraries

```bash
echo "/usr/lib/jvm/jdk-21-oracle-x64/lib" > /etc/ld.so.conf.d/oracle-jdk21.conf
echo "/usr/lib/jvm/jdk-21-oracle-x64/lib/server" >> /etc/ld.so.conf.d/oracle-jdk21.conf
ldconfig
```

### Step 3: Restart and Verify

```bash
systemctl reset-failed tomcat
systemctl restart tomcat
ss -tulnp | grep java
```

**Verification result:**
```
tcp   LISTEN   0   100   *:80       *:*   users:(("java",pid=485124))
tcp   LISTEN   0   100   *:443      *:*   users:(("java",pid=485124))
tcp   LISTEN   0   1     127.0.0.1:8005   *:*   users:(("java",pid=485124))
```

---

## Recommendations

1. **Re-apply setcap after JDK upgrades** — Capabilities are stored on the binary's inode and lost when the file is replaced.
2. **Consider reverse proxy** — Deploy nginx or HAProxy on ports 80/443 forwarding to Tomcat on 8080/8443 to avoid capability edge cases.
3. **Implement health monitoring** — Add periodic `curl` checks to catch silent connector failures early.
4. **Document in runbook** — Include capability re-application in JDK upgrade procedures.
5. **Test as target user** — Always verify library resolution as the non-root user after applying capabilities.

---

## Technical Reference

### AT_SECURE Mode

When a binary has capabilities, setuid, or setgid bits set, the kernel activates AT_SECURE mode for non-root users. This causes glibc's dynamic linker to:

- Ignore `LD_LIBRARY_PATH`
- Ignore `RUNPATH` entries
- Only search `/lib`, `/usr/lib`, and `/etc/ld.so.cache`

### Why Root Is Exempt

Root (UID 0) bypasses AT_SECURE restrictions for backward compatibility. This is why the same binary works as root but fails as a non-root user.

### Capability Persistence

Capabilities are stored in the file's extended attributes (xattrs) on the inode. They persist across:
- Service restarts
- System reboots
- File moves (within same filesystem)

They are lost when:
- The file is replaced (e.g., JDK upgrade)
- The file system is reformatted
- Extended attributes are explicitly removed

---

## Appendix: Commands Used

```bash
# Diagnostics
ps -ef | grep java
ss -tulnp | grep java
systemctl show tomcat -p AmbientCapabilities
getcap /usr/lib/jvm/jdk-21-oracle-x64/bin/java
ldd /usr/lib/jvm/jdk-21-oracle-x64/bin/java

# Fix
setcap 'cap_net_bind_service=+ep' /usr/lib/jvm/jdk-21-oracle-x64/bin/java
echo "/usr/lib/jvm/jdk-21-oracle-x64/lib" > /etc/ld.so.conf.d/oracle-jdk21.conf
echo "/usr/lib/jvm/jdk-21-oracle-x64/lib/server" >> /etc/ld.so.conf.d/oracle-jdk21.conf
ldconfig
systemctl reset-failed tomcat
systemctl restart tomcat

# Verification
curl -I http://localhost:80/
curl -Ik https://localhost:443/
ss -tulnp | grep java
```

---



