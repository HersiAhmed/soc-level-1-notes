# Cyber Kill Chain

The Cyber Kill Chain (Lockheed Martin, 2011) breaks an attack into **7 stages in order**. The idea comes from the military: **break any link in the chain and the attack fails**. The earlier you break it, the less damage there is.

| # | Phase | The attacker... | Where a defender can step in |
|---|---|---|---|
| 1 | Reconnaissance | Researches the target | Limit public info, watch for port scans |
| 2 | Weaponization | Builds or buys the attack kit | Threat intel (this happens on the attacker's side) |
| 3 | Delivery | Gets the weapon to the victim | Email gateway, web proxy, user training |
| 4 | Exploitation | Gets their code to run | Patching, macro blocking, EDR |
| 5 | Installation | Sets up a way back in | EDR, Sysmon, service creation logs |
| 6 | Command & Control | Malware checks in with the attacker | Network monitoring, DNS logs, IDS |
| 7 | Actions on Objectives | Steals, spreads, or destroys | Detect exfiltration, isolate hosts, backups |

## 1. Reconnaissance

The attacker's homework: employees, email addresses, systems, and exposed technology.

- **OSINT (open-source intelligence):** info anyone can find. LinkedIn, company sites, WHOIS, search engines.
- **Passive recon:** no contact with the target (WHOIS, social media). Invisible to defenders.
- **Active recon:** touches the target (port scanning, banner grabbing, calling the help desk). Can show up in firewall or IDS logs, often the earliest warning sign you'll get.
- **Email harvesting:** collecting addresses for phishing (tools: theHarvester, Hunter.io). Once the format is known (`first.last@company.com`), anyone on LinkedIn can be targeted.

## 2. Weaponization

Building or buying the weapon. Most attackers use automated tools or buy malware on the dark web. **APTs** (well-funded, often government-backed groups) write custom malware, so no hash or signature exists for it yet.

Terms I keep straight with a burglary analogy:

| Term | Meaning | Analogy |
|---|---|---|
| **Exploit** | Code that abuses a flaw | The lockpick |
| **Payload** | Code that runs once inside | What the burglar does once in |
| **Malware** | Any harmful software | Burglary tools in general |

Examples: macro-enabled Office documents, infected USB drives, backdoors, setting up C2 servers, fake **OAuth consent apps** (a real "Allow access?" popup that hands over account access without the password, so MFA doesn't help).

## 3. Delivery

- **Phishing** (many people) vs **spear phishing** (one person, personalized using recon)
- **USB drops**, including drives printed with the company's logo and mailed as a "gift"
- **Watering hole:** hack a website the targets already visit and wait for them (like a lion waiting at the watering hole). Can lead to a **drive-by download**, where malware downloads just from visiting a page.

This is usually the first phase defenders can actually see, and phishing alerts are a big part of SOC work.

## 4. Exploitation

The moment the attacker's code runs.

| | Zero-day | Known CVE |
|---|---|---|
| Public? | No | Yes |
| Patch exists? | No | Usually |

Most real breaches use known, unpatched CVEs, not zero-days. After getting in, attackers usually escalate privileges and move laterally.

**Signs to look for:** unexpected process launches (Word → PowerShell), registry changes or new services, suspicious command lines.

## 5. Installation (persistence)

Planting a way back in that survives reboots, patches, and cleanups:

| Method | MITRE ATT&CK | How to catch it |
|---|---|---|
| **Web shell** (small `.php`/`.aspx` script on a web server) | | Search web folders for code like `eval`, `base64_decode`, `shell_exec` |
| **Backdoor** (e.g. Meterpreter) | | EDR |
| **Malicious Windows service** | T1543.003 | Event ID 7045 (new service installed) |
| **Registry run keys / Startup folder** | T1547.001 | Registry monitoring (Sysmon, EDR) |
| **Masquerading** (legit-sounding names) | T1036 | Check publisher, path, and parent process |
| **Timestomping** (faking file timestamps) | T1070.006 | Compare against other evidence |

If you remove the malware but miss the persistence, the attacker comes back.

## 6. Command & Control

The malware **beacons**: checks in with the attacker on a regular schedule.

- **IRC:** the old way, now easy to spot on a business network
- **HTTP/HTTPS (80/443):** blends in with normal web traffic
- **DNS tunneling:** data hidden in DNS lookups, like `aGVsbG8.evil-site.com`. Red flag: lots of requests to one domain with long random subdomains

Cut C2 and the malware is stranded with no orders.

## 7. Actions on Objectives

Stealing credentials, escalating to **domain admin**, internal recon, lateral movement, collecting and exfiltrating data, deleting **shadow copies** (Windows backups, so the victim can't recover), and encrypting or destroying data.

## Practice: the Target breach (2013)

The room ended with mapping the Target breach (about 40 million payment cards stolen) to the Kill Chain. The real story: attackers phished a **third-party HVAC vendor**, used the vendor's stolen login to get into Target's network, moved to the **point-of-sale systems**, and installed malware that collected card data as customers paid.

Big lesson: the way in was a **small vendor with weaker security**, not Target itself. Third-party risk is also the focus of my master's project on K-12 student data.
