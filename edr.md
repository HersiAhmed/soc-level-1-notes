# Endpoint Detection and Response (EDR)

## Why EDR exists

Companies used to protect the **network perimeter**, like a wall around a castle. With remote work, laptops now sit at home and in coffee shops, outside that wall. EDR protects each device wherever it is.

Examples: CrowdStrike Falcon, SentinelOne, Microsoft Defender for Endpoint, Symantec EDR.

## The three pillars

1. **Visibility.** Records process activity, registry changes, file changes, user actions, and network connections. Shows them as a **process tree** so you can see which program launched which.
2. **Detection.** Signatures *and* behavior. Uses machine learning to flag deviations from normal, catches fileless malware in memory, and accepts custom IOCs.
3. **Response.** From one console: isolate a host, kill a process, quarantine a file, or open a remote shell on the machine.

One limit: EDR is **host-only**. It doesn't see network-level threats, so it still needs firewalls, network monitoring, and email security around it.

## EDR vs antivirus

Antivirus is like **passport control at an airport**. It checks people against a list of known criminals. A thief with a clean record walks right through.

EDR is like **security staff watching cameras inside the airport**. They don't care who you are; they care what you *do*.

**Example attack chain: a phishing Word document**

| Step | Antivirus | EDR |
|---|---|---|
| User opens the Word doc | Word is legit, so nothing | Logs it |
| Hidden macro runs | No known signature, so nothing | Logs it |
| Word launches PowerShell | Both are legit, so nothing | **Flags the unusual parent-child relationship** |
| Obfuscated PowerShell downloads a payload | Usually missed | Flags the obfuscated script |
| Payload injected into `svchost.exe` | Doesn't watch memory | Detects process injection |
| Attacker gets remote access | No network visibility | Flags `svchost.exe` making an odd outbound connection |

Every step looks innocent alone, which is why antivirus misses it. EDR catches it by looking at the **chain of behavior**.

In practice, most modern EDRs include antivirus-style blocking built in, so it's less "EDR instead of AV" and more "EDR includes AV."

## How it works: agents and console

- **Agent (sensor):** installed on every endpoint. Collects activity data and sends it in real time. These are the eyes and ears.
- **Console:** receives data from all agents, correlates it with logic, machine learning, and threat intel, and turns it into **alerts**. This is the brain.

## Telemetry

Telemetry is the data agents collect. I think of it as an **airplane's black box**: it records everything so investigators can replay what happened.

| Telemetry | Helps detect |
|---|---|
| Process executions and terminations | Suspicious parent-child processes (Word → PowerShell) |
| Network connections | C2 traffic, data exfiltration, lateral movement, odd ports |
| Command line activity | Malicious or obfuscated CMD/PowerShell commands |
| File and folder modifications | Ransomware, dropped malware, data staging |
| Registry modifications | Persistence (malware set to auto-start) |

## Detection techniques

| Technique | The question it asks |
|---|---|
| Behavioral | What is it *doing*? |
| Anomaly | Is this normal *for this machine*? (can cause false positives) |
| IOC matching | Is it on a known-bad list? (misses zero-days) |
| MITRE ATT&CK mapping | Which attack stage is this? |
| Machine learning | Does the whole chain look like an attack? (fileless, multi-stage) |

**MITRE ATT&CK** is a public encyclopedia of real attacker behavior. **Tactics** are the attacker's goal (Persistence, Lateral Movement, Exfiltration…). **Techniques** are how they do it (e.g. Scheduled Task/Job, T1053).

## Response actions (smallest to biggest)

1. **Quarantine** a file, moving it somewhere it can't run
2. **Terminate** a process, when isolating the whole host would hurt the business
3. **Isolate** the host from the network, to stop lateral movement
4. **Remote access**, for custom investigation or actions
5. **Artifact collection**: memory dumps, event logs, folders, registry hives for forensics

## What I'd say in an interview

> EDR agents collect telemetry from each endpoint and the console correlates it into alerts. It catches what antivirus misses because it looks at behavior chains, like Word spawning PowerShell, instead of just signatures. From one console I can quarantine, kill a process, or isolate a host.
