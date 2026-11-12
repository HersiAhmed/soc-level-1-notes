# MITRE Frameworks

MITRE is a US not-for-profit that does research for the government and public. In cybersecurity it runs CVE.org and created a set of free frameworks used by both attackers (red teams) and defenders (blue teams).

| Framework | Answers | Memory trick |
|---|---|---|
| **ATT&CK** | How do attackers attack? | Offense |
| **CAR** | How do I detect it? | Detection |
| **D3FEND** | How do I defend against it? | Defense |
| **Engage** | How do I deceive and study attackers? | Deception |

## TTPs

| Part | Question | Bank robbery example |
|---|---|---|
| **Tactic** | Why? (the goal) | Get into the vault |
| **Technique** | How? | Pick the lock |
| **Procedure** | Exactly how this group does it | This crew uses a specific lockpick set at 3am after cutting the alarm |

TTPs sit at the top of the Pyramid of Pain: attackers can swap IPs and hashes easily, but changing how they operate is hard.

## ATT&CK

### Reading the matrix

- **Columns** = tactics (Reconnaissance, Initial Access, Execution, Persistence...)
- **Boxes** = techniques
- **Grey bar on a box** = it has sub-techniques

### ID formats

| Prefix | Meaning | Example |
|---|---|---|
| `TA` | Tactic | TA0011 Command and Control |
| `T` | Technique | T1595 Active Scanning |
| `T....###` | Sub-technique | T1595.002 Vulnerability Scanning |
| `G` | Group | G0129 Mustang Panda |
| `S` | Software/malware | |

### A technique page includes

Description, platforms, the tactic it belongs to, **procedure examples** (which groups used it and how), **mitigations**, **detection**, and references. For SOC work, mitigations and detection are the most useful parts.

### Matrices

**Enterprise** (Windows, macOS, Linux, cloud), **Mobile**, and **ICS** (industrial control systems like power plants).

### Who uses it

| Role | How |
|---|---|
| CTI teams | Profile attacker groups by their TTPs |
| SOC analysts | Understand what stage an alert represents |
| Detection engineers | Map rules to techniques to find coverage gaps |
| Incident responders | Map the attack timeline |
| Red and **purple** teams | Emulate real groups to test defenses (purple = red and blue working together) |

It also gives everyone **one shared language**. "T1053" means the same thing to every company.

### Example: Mustang Panda (G0129)

Targets governments, non-profits, and NGOs. Known techniques include phishing for initial access, scheduled tasks for persistence, obfuscated files for defense evasion, and ingress tool transfer (downloading more tools) for C2. If a group like this targets your sector, you make sure you can detect those specific techniques.

## CAR (Cyber Analytics Repository)

Ready-made detection logic for ATT&CK techniques, with pseudocode and real queries.

**Example: CAR-2020-09-001, Scheduled Task – File Access**

When a scheduled task is created, Windows writes a file to `C:\Windows\System32\Tasks` (or `C:\Windows\Tasks`). The Splunk version:

```spl
index=__your_sysmon_index__ EventCode=11
Image!="C:\\WINDOWS\\system32\\svchost.exe"
(TargetFilename="C:\\Windows\\System32\\Tasks\\*" OR TargetFilename="C:\\Windows\\Tasks\\*")
```

| Part | Meaning |
|---|---|
| `EventCode=11` | Sysmon Event 11 = file created |
| `Image!=...svchost.exe` | Skip files created by the legit Task Scheduler service (false positive tuning built in) |
| `TargetFilename=...` | The new file is in a Tasks folder |

Coverage is marked **Low**: it catches some abuse of scheduled tasks, not all. I plan to use this exact query in my home lab.

## D3FEND

ATT&CK's defensive mirror. IDs start with `D3-`. Its 7 tactics:

| Tactic | Example |
|---|---|
| Model | Asset inventory, threat modelling |
| Harden | Patching, MFA |
| Detect | SIEM rules, EDR |
| Isolate | Isolate a host, network segmentation |
| Deceive | Honeypots |
| Evict | Kill processes, disable stolen accounts |
| Restore | Restore from backups |

**Example: Credential Rotation (D3-CRO).** Regularly changing passwords, keys, and certificates so stolen credentials stop working. It directly counters credential theft followed by Pass-the-Hash.

## Other MITRE projects

| Project | Purpose |
|---|---|
| Adversary Emulation Library | Step-by-step plans copying real groups |
| Caldera | Automatically runs ATT&CK techniques to test defenses |
| AADAPT | ATT&CK-style matrix for crypto and blockchain |
| ATLAS | ATT&CK-style matrix for AI/ML (prompt injection, data poisoning) |

## How I worked through the threat-hunting exercise

The final exercise followed an analyst tracing an APT through its ATT&CK Navigator layer. The method that worked every time:

1. Find the **tactic** word in the question → that's the column
2. Find the **technique** word → that's the box
3. Expand the sub-techniques; the **highlighted** ones are what the group is known to use

The questions followed the attack in order (recon → resource development → initial access → execution → persistence → defense evasion → discovery → lateral movement → collection → C2). That's how CTI becomes action: "this group might attack us" turns into "here's exactly what to search our logs for."
