# Security Information and Event Management (SIEM)

## What a SIEM is

A SIEM is the **one central place** a SOC analyst watches. It collects logs from *everything* (endpoints, servers, firewalls, email security, identity systems, EDR), puts them in one format, connects related events, and raises alerts.

If EDR is the cameras inside one building, the SIEM is the **control room watching every camera, door alarm, and badge reader on the whole campus**.

## Log sources

| Type | What it captures | Examples |
|---|---|---|
| **Host-centric** | What happens *on* a device | File access, logins, process execution, registry changes, PowerShell |
| **Network-centric** | Devices talking to each other or the internet | SSH, FTP, web traffic, VPN, file sharing (from firewalls, routers, IDS/IPS) |

Quick test: inside one machine → host-centric. Traffic between machines → network-centric.

## Why isolated logs don't work

1. **Too many sources.** Hundreds of events per second across dozens of devices.
2. **No centralization.** You'd have to SSH/RDP into each machine.
3. **Limited context.** A file access looks normal until you see the user got there through lateral movement.
4. **Limited analysis.** No human can read millions of logs.
5. **Format issues.** Every device writes logs differently.

## Core features

**Centralized collection.** Logs flow in through agents/forwarders or APIs.

**Parsing vs normalization.** I mixed these up at first, so here's the difference:

- **Parsing = splitting.** Breaks one raw log into labeled fields.
  `Oct 13 09:02 haris login success 10.0.0.5` → `Time` | `User=haris` | `Action=login` | `Result=success` | `IP=10.0.0.5`
- **Normalization = matching.** Makes every source use the same field names. Windows says `AccountName`, Linux says `user`, and both become `User`.

**Correlation.** Connecting events across sources. Example, all within 5 minutes:

- VPN login from an IP the user has never used
- Opens documents on a shared drive
- Runs a PowerShell script
- Machine makes an outbound connection

Each one alone looks fine. Together, they look like stolen VPN credentials followed by **data exfiltration**.

**Real-time alerting.** Detection rules fire alerts. **Dashboards** summarize alerts, failed logins, ingestion counts, top domains, and so on.

## Where logs live

**Windows:** Event Viewer. Every event type has an **Event ID**. The Security log matters most.

**Linux:** text files under `/var/log/`

| File | Contents |
|---|---|
| `/var/log/auth.log`, `/var/log/secure` | Authentication |
| `/var/log/httpd`, `/var/log/apache` | Web requests and errors |
| `/var/log/cron` | Scheduled jobs |
| `/var/log/kern` | Kernel events |

**Reading an Apache log line:**

```
192.168.21.200 - - [21/March/2022:10:17:10 -0300] "GET /cgi-bin/try/ HTTP/1.0" 200 3395 "-" "Mozilla/5.0 ... Chrome/98..."
```

Who (IP) → when → what was requested (`GET /cgi-bin/try/`) → result (`200` = success) → size → user agent (browser). A user agent of `curl` instead of a browser often means a script, not a normal visitor.

## Ingestion methods

1. **Agent / forwarder** on each device
2. **Syslog**, a standard protocol for shipping logs in real time
3. **Manual upload** for one-off analysis
4. **Port forwarding**, where the SIEM listens on a port and devices send to it

## Detection rules

A rule is an **IF / THEN** condition on log fields.

| Rule | Why it matters |
|---|---|
| 5 failed logins in 10 seconds | Brute force attempt |
| Successful login after many failures | The brute force worked |
| USB plugged in (where banned) | Possible data theft or malware |
| Outbound traffic over a threshold | Possible exfiltration |

**Example: attacker clearing logs to hide their tracks**

```
IF log_source = WinEventLog AND EventID = 104
THEN alert "Event Log Cleared"
```

**Example: attacker checking their privileges with `whoami`**

```
IF log_source = WinEventLog AND EventCode = 4688 AND NewProcessName contains "whoami"
THEN alert "WHOAMI command execution"
```

These only work because logs were normalized into consistent fields first.

## Handling an alert

1. Look at the events behind the alert and which rule conditions matched
2. **False positive?** Tune the rule (e.g. exclude an admin who legitimately runs `whoami`)
3. **True positive?** Investigate further, contact the asset owner, isolate the host, block the IP

## SIEM vs EDR

| | EDR | SIEM |
|---|---|---|
| Watches | Devices | Everything |
| Main job | Detect and act on a device | Collect, correlate, alert |
| Can isolate a host itself? | Yes | No |
