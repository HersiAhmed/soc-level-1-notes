# Pyramid of Pain

The Pyramid of Pain ranks indicators by **how much pain it causes an attacker when you detect and block them**. The bottom is easy for an attacker to change. The top forces them to rethink how they attack.

```
                 ▲
                / \        TTPs                    Tough
               /───\       Tools                   Challenging
              /─────\      Network/Host Artifacts  Annoying
             /───────\     Domain Names            Simple
            /─────────\    IP Addresses            Easy
           /───────────\   Hash Values             Trivial
```

The higher you detect, the more likely the attacker gives up and moves to an easier target.

## 1. Hash values (trivial)

A hash is a **fingerprint of a file**: a fixed-length string calculated from its contents.

| Algorithm | Hex characters | Still secure? |
|---|---|---|
| MD5 | 32 | No (collisions found) |
| SHA-1 | 40 | No (deprecated) |
| SHA-256 | 64 | Yes |

Trick I use: count the characters to tell which algorithm a hash is.

**How analysts use them:** look the hash up on VirusTotal or MetaDefender, then search for it across endpoints with the SIEM or EDR.

**Why they're weak:** change a file by a single byte and the hash is completely different.

```powershell
Get-FileHash .\installer.msi -Algorithm MD5      # hash A
echo "AppendTheHash" >> .\installer.msi            # >> appends, > would overwrite
Get-FileHash .\installer.msi -Algorithm MD5      # totally different hash B
```

The file still works the same, but now no blocklist knows it.

## 2. IP addresses (easy)

Blocking an attacker's IP on the firewall is fast and stops unsophisticated attackers, but they can switch to a new IP in seconds.

**Fast Flux** makes IP blocking close to useless. This took me a few tries to get, so here's how I'd explain it:

1. The malware has one **domain** baked in, e.g. `evil-site.com`. It doesn't carry a list of IPs.
2. To reach its C2 server, it asks DNS "what IP is `evil-site.com`?"
3. The attacker keeps changing the DNS answer, pointing it at **hacked computers belonging to innocent people** (a botnet).
4. Those machines just relay the traffic to the real C2 server, which never appears in your logs.

```
10:00  evil-site.com → 1.1.1.1  (hacked PC, Brazil)
10:05  evil-site.com → 2.2.2.2  (hacked PC, Poland)
10:10  evil-site.com → 3.3.3.3  (hacked PC, Japan)

Malware → hacked relay (constantly changing) → real C2 server (hidden)
```

Block one IP and the malware moves to the next. What *doesn't* change is the domain, which is why the next level matters.

## 3. Domain names (simple)

A new domain takes more effort than a new IP (buy it, register it, set up DNS), but it's still not hard, especially with providers that offer APIs.

**Malicious domains often look normal.** Ransomware C2 lists are full of what look like small business sites, many of them legitimate sites that were hacked.

**Punycode attacks.** Domains are limited to ASCII, so Punycode converts other characters (accents, other alphabets) into ASCII, always starting with `xn--`. Attackers abuse lookalike characters:

| Looks like | Actually is |
|---|---|
| `adidas.de` | `adıdas.de` (Turkish dotless ı) → `xn--addas-o4a.de` |

To a person it reads the same. To a computer it's a completely different domain. `xn--` in proxy logs is worth a second look.

**URL shorteners** (bit.ly, tinyurl, etc.) hide the real destination. Ways to check without visiting:

- Add `+` to the end of a bit.ly link
- Use the service's preview page (e.g. `preview.tinyurl.com/...`)
- Watch for **stacked redirects**: the real destination can be URL-encoded inside another redirect. Decode it (`%3A` = `:`, `%2F` = `/`) and stop at the `&`.

## Reading a sandbox report (Any.run)

Any.run runs malware in an isolated VM and records everything it does. Things I look at:

- **Summary:** file name, hashes, tags (e.g. ransomware family)
- **Process tree:** what launched what. A ransomware sample's tree showed it deleting the victim's recovery options before encrypting:

```
malware.exe
 └─ cmd.exe /c vssadmin ... & bcdedit ...
     ├─ vssadmin.exe Delete Shadows /All /Quiet          ← deletes backup copies
     ├─ bcdedit.exe /set {default} recoveryenabled No    ← disables recovery mode
     └─ bcdedit.exe /set {default} bootstatuspolicy ignoreallfailures
```

`vssadmin Delete Shadows` is a classic ransomware signal and a good detection rule.

- **Network tabs:**

| Tab | Useful for |
|---|---|
| HTTP Requests | Spotting a dropper downloading the next stage |
| Connections | C2 traffic, file transfers, odd ports |
| DNS Requests | Which domains the malware looks up (often first, to check if it's online or in a sandbox) |

Never visit IPs or URLs from a malware report.

## 4. Host artifacts (annoying)

Traces an attack leaves **on the machine**: suspicious processes, dropped files, registry changes. This is what EDR telemetry collects.

**Example chain from the room:**

```
Malicious Word doc opened
  → WINWORD.EXE launches a disguised program
     → POwersheLL.exe downloads G_jugk.exe from an external IP
        → the same file (identical hash) is copied to AppData\...\regidle.exe to hide
```

**How I'd spot the disguised program Word launched:**

1. **Parent:** Word shouldn't launch other programs
2. **Name:** mimics a Windows system DLL (`api-ms-win-...`), but real ones are libraries, not running processes
3. **Description mismatch:** the name says "Windows file," the description says "EffectDemo MFC Application"
4. **No publisher:** Word shows Microsoft Corporation; this one is blank

Also note `POwersheLL.exe`: odd capitalization to dodge case-sensitive detection rules. Rules should ignore case.

Detecting this behavior keeps working even after the attacker swaps their IP, domain, and hash.

## 5. Network artifacts (annoying)

Traces an attack leaves **in network traffic**:

- **User-Agent strings**, the "name tag" a program sends with web requests
- **URI patterns** (the path part of a URL)
- **C2 behavior**, like beaconing (regular check-ins)

**Suspicious pattern example:** one internal host sending repeated `POST` requests (sending data, not fetching pages) to many different external IPs, with random gibberish paths like `/Nqdlz/w2BG/`. Readable paths like `/login` are normal; random ones are often generated by malware.

**Checking User-Agents in a PCAP with TShark:**

```bash
tshark -Y http.request -T fields -e http.host -e http.user_agent -r capture.pcap
```

| Part | Meaning |
|---|---|
| `-Y http.request` | Only HTTP requests |
| `-T fields -e ...` | Print only these fields |
| `-r capture.pcap` | Read this capture file |

Emotet traffic showed the same User-Agent on every request, claiming to be Internet Explorer 7 on Windows 7. That's an ancient browser that no real user runs today, hardcoded into the malware. Blocking it forces the attacker to change their code, not just their server.

**Host vs network indicators**

| | Found | Seen by |
|---|---|---|
| Host | On the computer (files, processes, registry) | EDR |
| Network | In traffic (IPs, domains, User-Agents) | Firewall, IDS (Snort), proxy logs, PCAPs |

## 6. Tools (challenging)

The software the attacker uses: maldocs for spearphishing, backdoors, custom EXEs and DLLs, payloads, password crackers. Detecting the tool itself means the attacker has to build, buy, or learn a new one.

**Weapons at this level:**

- **Antivirus signatures** and **detection rules** (SIEM/EDR, Sigma)
- **YARA rules**, which match patterns *inside* files (strings, byte sequences)
- **Fuzzy hashing**

**Fuzzy hashing (SSDEEP)** fixes the weakness of normal hashes. It splits a file into chunks and hashes each one, so a small change only affects nearby chunks.

| | Normal hash (MD5/SHA-256) | Fuzzy hash (SSDEEP) |
|---|---|---|
| Changes one byte | Completely different | Almost the same |
| Answers | "Is this the exact same file?" | "Is this a close variant?" (score 0–100) |

A normal hash is a fingerprint; a fuzzy hash is a family resemblance. It catches the attacker's tool even after they tweak it.

**Useful resources:** MalwareBazaar and MalShare (samples, YARA results), SOC Prime Threat Detection Marketplace (shared detection rules).

## 7. TTPs (tough)

**Tactics, Techniques, and Procedures**: how the attacker operates. This is what MITRE ATT&CK catalogs.

| Part | Meaning | Example |
|---|---|---|
| Tactic | The goal | Lateral Movement |
| Technique | How they reach it | Pass-the-Hash |
| Procedure | One group's exact steps | Dump an admin hash, then use it to log into the file server |

**Pass-the-Hash:** the attacker steals a password *hash* and uses it to log in directly, without ever knowing the password. It's like copying a key card instead of learning the PIN. They use it to spread to other machines.

**Detection:** Windows Event Log monitoring for unusual logon patterns (e.g. Event ID 4624 for one account suddenly showing up across many machines). Catching it means finding the compromised host and stopping lateral movement early.

An attacker can change everything lower on the pyramid. Changing their TTPs means learning a new way to attack, so most move on to an easier target instead.

## Summary

| Level | Beaten by |
|---|---|
| Hashes | Changing one byte |
| IPs | Switching servers (Fast Flux) |
| Domains | New domains, Punycode lookalikes, shorteners |
| Artifacts | Changing how the malware behaves |
| Tools | Building or buying new tools |
| TTPs | Learning a whole new way to attack |
