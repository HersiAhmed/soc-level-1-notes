# Unified Kill Chain (UKC)

The Unified Kill Chain (Paul Pols, 2017, updated 2022) is a more detailed, modern version of the Cyber Kill Chain. It combines ideas from the original Kill Chain and MITRE ATT&CK.

## Why it exists

| Cyber Kill Chain | Unified Kill Chain |
|---|---|
| 7 phases | **18 phases** |
| Mostly about getting in | Covers the **whole attack**, including what happens inside and the attacker's motive |
| A straight line | Recognizes that attackers **loop back** and repeat phases |

After breaking into one machine, real attackers do recon again, escalate, and move to the next machine. The UKC reflects that cycle.

## Threat modelling

The room starts with threat modelling: thinking through how something could be attacked before it happens.

1. Identify what needs protecting and how important it is
2. Find its weaknesses
3. Plan fixes
4. Add policies so the problem doesn't come back (e.g. secure development, phishing training)

Frameworks: **STRIDE** (6 threat types), **DREAD** (scoring threats), **CVSS** (the 0–10 severity score attached to CVEs).

My master's project is essentially a threat model for K-12 student data: tier the data by sensitivity, analyze real breaches for weaknesses, write requirements, and require security reviews at every contract renewal.

## The 18 phases in 3 stages

### IN: getting a foothold

| Phase | Meaning |
|---|---|
| Reconnaissance | Research the target |
| Weaponization | Set up tools and infrastructure, e.g. a server to catch **reverse shells** (the victim connects *out* to the attacker, since firewalls usually allow outgoing traffic) |
| Delivery | Get the weapon to the victim |
| **Social Engineering** | Manipulate people: phishing, fake login pages, phone impersonation |
| Exploitation | Abuse a flaw to run code |
| Persistence | Keep access |
| **Defense Evasion** | Get past firewalls, WAFs, antivirus, IDS. Valuable for defenders, because it shows what to improve |
| Command & Control | Remote control of the victim |

### THROUGH: moving around inside

| Phase | Meaning |
|---|---|
| **Pivoting** | Use the hacked machine as a stepping stone to systems not reachable from the internet |
| **Discovery** | Internal recon: accounts, permissions, software, network shares |
| Privilege Escalation | SYSTEM/root > local admin > admin-like user > user with specific access |
| Execution | Run more code from the pivot machine (trojans, scheduled tasks) |
| Credential Access | Steal passwords and hashes (keylogging, credential dumping, e.g. Mimikatz). With real logins, the attacker looks like a normal employee |
| Lateral Movement | Hop to other machines (e.g. Pass-the-Hash) |

**Pivoting vs lateral movement:** pivoting routes traffic *through* a hacked machine; lateral movement *takes control* of other machines.

These phases loop: discover → escalate/steal credentials → move → discover again.

### OUT: achieving the goal

Framed around the **CIA triad** (confidentiality, integrity, availability):

| Phase | Hits | Meaning |
|---|---|---|
| Collection | Confidentiality | Gather the data (drives, browsers, email, even audio/video) |
| Exfiltration | Confidentiality | Send it out, usually compressed and encrypted, through the C2 channel |
| Impact | Integrity, Availability | Ransomware, disk wipes, locking accounts, defacement, DoS |
| **Objectives** | | The motive: money, reputation damage, espionage |

**Double extortion:** modern ransomware gangs steal the data *and* encrypt it, then threaten to leak it even if the victim restores from backups.

## What defenders watch for

| Phase | Red flags |
|---|---|
| Discovery | `whoami`, `net user`, `net share`; Event ID 4688 |
| Privilege Escalation | Accounts suddenly added to admin groups |
| Credential Access | A process reading password memory |
| Lateral Movement | One account logging into many machines it never uses (4624) |
| Collection / Exfiltration | Lots of files zipped into one place; large outbound transfers |
| Impact | `vssadmin Delete Shadows`, mass file renaming |

## UKC vs MITRE ATT&CK

They complement each other. The UKC tells the **story** of an attack phase by phase. ATT&CK is the **detailed encyclopedia** of techniques used in each phase.
