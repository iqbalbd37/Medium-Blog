### *Troubleshooting Oracle Database Connection Timeouts in Spring Boot and Apache Tomcat: A DevOps Incident Analysis*

---

> *A practical, production-oriented guide to diagnosing Oracle JDBC connectivity failures, HikariCP initialization errors, Hibernate dialect exceptions, and Tomcat port conflicts on Linux.*

---

### Introduction

In production environments, an application can fail to deploy even when its Java runtime, application archive, and Tomcat installation appear to be functioning correctly.

One common reason is a failure in the application-to-database communication path.

During a recent log investigation, a Java web application deployed on Apache Tomcat repeatedly failed to establish a connection to an Oracle database. The logs contained several errors:

- `SQLRecoverableException`
- `The Network Adapter could not establish the connection`
- `java.net.ConnectException: Connection timed out`
- `Unable to determine Dialect without JDBC metadata`
- `java.net.BindException: Address already in use`

At first glance, these messages might appear to indicate several unrelated problems. However, careful log analysis helps distinguish the primary failure from its downstream symptoms and the additional Tomcat configuration issue.

This article walks through the investigation from a Linux and DevOps operations perspective, using anonymized infrastructure details.

---

### 1. Understanding the Application Environment

The captured logs identify the following software components:

| Component | Version or detail |
|---|---|
| Operating System | Linux environment; distribution not established by the supplied log |
| Java | Java 21 runtime |
| Apache Tomcat | 10.1.15 |
| Spring Boot | 3.5.4 |
| Hibernate ORM | 6.4.4.Final |
| Oracle JDBC driver | ojdbc11 21.9.0.0 |
| HikariCP | 6.3.1 |
| Oracle listener port | TCP 1521 |
| Tomcat shutdown port | TCP 8005 |

The Oracle database IP address has been anonymized to `10.20.30.40`. This is a documentation placeholder, not the original database address or a claim that this address is configured in the environment.

The logs indicate that the application is deployed under a Tomcat web application directory. Spring Boot initializes JPA, Hibernate, and HikariCP as part of the deployment process.

If the database connection cannot be established during this initialization, the application may fail before it becomes ready to serve requests.

---

### 2. Identifying the Primary Error

The most important error in the log is:

```text
java.sql.SQLRecoverableException:
IO Error: The Network Adapter could not establish the connection
```

The underlying exception provides more specific evidence:

```text
Caused by: java.net.ConnectException:
Connection timed out
```

The connection attempt targets the Oracle listener on TCP port `1521`.

#### What does this mean?

The application attempted to establish a TCP connection to the configured database endpoint but did not receive a successful connection before the attempt timed out.

This is different from an Oracle authentication error.

For example, an invalid database password would typically produce an Oracle authentication-related error after the client reaches the database listener. Here, the failure occurs at the network connection stage.

Potential causes include:

1. The database host is unavailable.
2. The Oracle listener is stopped or not listening on the expected interface.
3. A firewall or network access-control rule is dropping traffic.
4. The application server has an incorrect database IP address or port.
5. A routing issue prevents communication between the application and database networks.
6. The database endpoint is undergoing maintenance or experiencing an infrastructure failure.

**Important:** A timeout does not prove which of these conditions is responsible. The available log establishes a failed TCP connection, not the exact network or database-side root cause.

---

### 3. Why HikariCP Cannot Create a Connection

The log shows HikariCP attempting to create a connection:

```text
HikariPool-1 - Starting...
HikariPool-1 - Attempting to create/setup new connection
```

The subsequent failure is:

```text
HikariPool-1 - Failed to create/setup connection
HikariPool-1 - Pool is empty, failed to create/setup connection
```

HikariCP is the JDBC connection pool used by the application. Rather than opening a completely new database connection for every operation, the application obtains connections from this pool.

During application startup, the pool attempts to establish physical connections to the configured Oracle endpoint.

When those attempts fail, HikariCP cannot provide a usable database connection.

The supplied configuration shows:

```text
maximumPoolSize.................6
minimumIdle.....................3
connectionTimeout...............600000
```

These settings describe a maximum pool size of six connections, a minimum idle target of three, and a configured connection acquisition timeout of 600,000 milliseconds.

However, increasing the pool size will not resolve an unreachable database. If the network connection itself cannot be established, more connection attempts may simply create additional load and longer startup delays.

#### Production Recommendation

Investigate basic network reachability before changing connection-pool settings.

Also review the unusually long connection timeout. A long timeout can make a startup failure slow to diagnose and delay recovery. Any adjustment should be based on the application's latency requirements, connection behavior, and operational standards.

---

### 4. Why Hibernate Reports a Dialect Error

Another exception appears during Hibernate initialization:

```text
org.hibernate.HibernateException:
Unable to determine Dialect without JDBC metadata
```

This message can be misleading if investigated in isolation.

Hibernate can often determine the appropriate SQL dialect from metadata obtained through a JDBC connection. In this incident, the application cannot obtain the required connection because Oracle connectivity has already failed.

The sequence is therefore:

1. Spring initializes the application's persistence configuration.
2. HikariCP attempts to create a JDBC connection.
3. The TCP connection to Oracle times out.
4. Hibernate cannot retrieve JDBC metadata.
5. Hibernate fails to determine the database dialect automatically.
6. The JPA initialization process fails, preventing successful application deployment.

The dialect exception is therefore consistent with the preceding database connection failure.

Explicitly configuring an Oracle dialect may remove the need for automatic dialect detection in some configurations, but it will not restore network connectivity or make database operations succeed.

**Troubleshooting principle:** Always investigate the earliest meaningful exception before attempting to fix later initialization errors.

---

### 5. Step-by-Step Linux Network Troubleshooting

Perform these checks from the application server, using an authorized account.

#### Step 1: Verify the configured endpoint

Inspect the application's configuration, environment variables, or externalized configuration files.

For Spring Boot, look for properties such as:

```properties
spring.datasource.url=jdbc:oracle:thin:@//10.20.30.40:1521/your_service
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

This is an illustrative JDBC URL format, not a replacement for the actual connection descriptor in the captured logs.

The supplied log uses an Oracle descriptor with a `SERVICE_NAME`. Preserve the correct connection format and verify the service name with the DBA.

Do not publish actual passwords, internal connection descriptors, or production secrets in a blog or public repository.

#### Step 2: Test TCP connectivity

On Linux, run:

```bash
nc -vz -w 5 10.20.30.40 1521
```

If Netcat is not installed, use an approved alternative such as:

```bash
timeout 5 bash -c '</dev/tcp/10.20.30.40/1521'
```

Interpret the results carefully:

- **Connection succeeded:** A TCP connection was established. Continue with Oracle listener, service, and JDBC validation.
- **Connection refused:** The host was reachable, but the connection was actively rejected. Check the listener and network security controls.
- **Connection timed out:** The connection was not established within the timeout. Investigate routing, firewalls, endpoint availability, and packet filtering.

A successful TCP test does not guarantee that the Oracle service is healthy or that application authentication will succeed.

#### Step 3: Check routing

Inspect the routing table:

```bash
ip route
```

Check how Linux routes traffic toward the database:

```bash
ip route get 10.20.30.40
```

Verify that the expected interface and gateway are being used.

If the application server and database are on separate subnets, confirm that the necessary routes exist in both directions.

#### Step 4: Inspect firewall and network controls

Check the local firewall according to the Linux distribution and firewall technology in use.

For systems using firewalld:

```bash
sudo firewall-cmd --state
sudo firewall-cmd --list-all
```

For systems using nftables:

```bash
sudo nft list ruleset
```

Do not disable the firewall as a troubleshooting shortcut.

Instead, ask the network or security team to verify that the application server's actual source IP is permitted to connect to the database endpoint on TCP port 1521.

Any rule change should follow the organization's change-management and security-approval process.

#### Step 5: Check the Oracle listener

On the database server, an authorized DBA can inspect listener status:

```bash
lsnrctl status
```

Verify that the listener is running and listening on the expected host and port.

Then check that the required database service is registered with the listener.

A running listener alone does not guarantee that the intended database service is available.

#### Step 6: Test Oracle connectivity independently

If an approved Oracle client is available on the application server, test connectivity using the DBA-provided service name and an authorized account.

For example:

```bash
sqlplus username@//10.20.30.40:1521/your_service
```

Enter credentials securely when prompted rather than placing passwords in shell history.

This test helps distinguish a general network problem from an application-specific JDBC configuration issue.

If the TCP connection works but the application still fails, inspect the JDBC URL, service name, credentials, driver compatibility, and application configuration.

---

### 6. A Second Problem: Tomcat Shutdown Port 8005 Is Already in Use

The log contains another important exception:

```text
SEVERE:
Failed to create server shutdown socket on address [localhost]
and port [8005]

java.net.BindException: Address already in use
```

This is separate from the Oracle connectivity problem.

Tomcat is configured to use a shutdown socket on port 8005, but it cannot bind that socket because the address and port combination is already occupied or otherwise unavailable for binding.

A common explanation is that another Tomcat process is already running with the same shutdown-port configuration.

Other possibilities include another process using the port or a process-management issue during a restart.

#### Step 1: Identify the process using port 8005

Run:

```bash
sudo ss -ltnp | grep ':8005'
```

Alternatively:

```bash
sudo lsof -nP -iTCP:8005 -sTCP:LISTEN
```

If a process is found, identify its command line and service ownership:

```bash
ps -fp <PID>
```

Replace `<PID>` with the process ID reported by the diagnostic command.

Do not kill the process until you confirm what it is, whether it serves live traffic, and whether it belongs to the intended Tomcat service.

#### Step 2: Check for duplicate Tomcat instances

Inspect running Java processes:

```bash
ps -ef | grep '[o]rg.apache.catalina.startup.Bootstrap'
```

Also inspect the systemd service, if applicable:

```bash
sudo systemctl status tomcat
```

Use the actual service name if it differs.

Review startup scripts, scheduled jobs, deployment automation, and service-manager configuration to determine whether Tomcat is being started twice.

#### Step 3: Review the Tomcat configuration

Inspect the Server element in:

```text
/opt/tomcat/conf/server.xml
```

The relevant configuration may look like:

```xml
<Server port="8005" shutdown="SHUTDOWN">
```

If the environment legitimately requires separate Tomcat instances on the same host, each instance must have a compatible and non-conflicting port configuration.

Alternatively, correct the duplicate-start mechanism if only one instance is intended.

Tomcat's shutdown port is a control interface, not an application HTTP port. If it is not required, an administrator may consider disabling it according to the deployed Tomcat version's supported configuration and operational procedures.

Do not change the port blindly: first identify the owning process and confirm the intended service design.

---

### 7. Recommended Resolution Order

For this incident, follow this sequence.

#### Priority 1 — Restore database reachability

- Verify the configured Oracle endpoint.
- Test TCP connectivity from the application host.
- Check routing and approved firewall rules.
- Confirm listener status and database service registration with the DBA.

#### Priority 2 — Resolve the Tomcat port conflict

- Identify the process occupying port 8005.
- Check for duplicate Tomcat instances.
- Correct service or deployment automation.
- Change the shutdown-port configuration only if the architecture requires it.

#### Priority 3 — Validate application initialization

After resolving the underlying issues:

- Restart Tomcat through the approved service-management procedure.
- Review the new application startup logs.
- Confirm that HikariCP establishes database connections.
- Confirm that Hibernate and JPA initialize successfully.
- Verify that the application is deployed and its health checks pass.

Do not assume that the application is healthy merely because Tomcat reports a server startup message. In this log, Tomcat reports an overall startup duration and subsequently encounters the shutdown-port error, while the web application's deployment has already failed.

---

### 8. Production-Grade Monitoring and Prevention

A reliable DevOps environment should detect database connectivity failures before they become prolonged application outages.

Consider implementing the following controls:

- **Application health checks:** Expose an appropriately secured health endpoint and distinguish process health from database readiness.
- **Network monitoring:** Monitor reachability between application and database networks.
- **Database monitoring:** Track Oracle listener availability, service registration, and database health.
- **Connection-pool metrics:** Monitor HikariCP active, idle, pending, and failed connection metrics.
- **Startup alerts:** Alert on repeated deployment failures and connection timeouts.
- **Process monitoring:** Detect duplicate Tomcat processes and unexpected listener-port conflicts.
- **Centralized logging:** Correlate application, operating system, firewall, and database events using timestamps.
- **Controlled recovery:** Use service-manager policies and deployment automation that prevent accidental duplicate starts.

For financial or other business-critical systems, validate recovery procedures in a non-production environment first. Avoid unplanned database restarts, broad firewall changes, or repeated production restarts without understanding the impact.

---

### 9. Security and Operational Lessons

This incident also highlights several operational best practices.

First, private IP addresses are still infrastructure information. Anonymize internal addresses, application names, service names, usernames, and connection identifiers before publishing logs.

Second, avoid sharing unmasked configuration dumps. HikariCP masks the password in the supplied log, but other logging configurations may expose sensitive values.

Third, keep database access restricted to authorized application hosts and required ports. Do not open Oracle port 1521 to unrestricted networks simply to make a connectivity test pass.

Finally, treat logs as evidence rather than assumptions. A timeout points to a failed connection attempt; it does not independently prove that the database is down, that a firewall is blocking traffic, or that the application configuration is incorrect.

---

### Conclusion

The captured incident demonstrates how one infrastructure failure can trigger several application-level exceptions.

The primary failure is an Oracle TCP connection timeout. This prevents HikariCP from creating a usable connection and causes Hibernate to fail when attempting to obtain JDBC metadata. Separately, Tomcat encounters a shutdown-port conflict on port 8005, resulting in an additional lifecycle problem.

The most effective troubleshooting strategy is to work from the bottom up: verify network connectivity, confirm Oracle listener availability, investigate Tomcat process and port ownership, and then validate application initialization.

**The key lesson for Linux administrators and DevOps engineers is simple: find the earliest meaningful failure, verify it with independent evidence, and fix the underlying cause before changing application settings.**

---

*Disclaimer: This case study is based on an anonymized application log. The precise infrastructure root cause has not been independently confirmed. The diagnostic commands and remediation steps are recommendations for investigation, not evidence that a specific fix was applied.*

---

