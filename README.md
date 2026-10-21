# SOC Level 1 Notes

My notes from working through the [TryHackMe SOC Level 1](https://tryhackme.com/path/outline/soclevel1) path. I hold CompTIA Security+ and I'm building toward a SOC Analyst I role.

These are written as **what I learned and how I'd explain it**, not as room answers, so there are no flags or spoilers here.

## Topics

| File | Topic | What it covers |
|---|---|---|
| [edr.md](01-edr.md) | Endpoint Detection and Response | Agents vs console, telemetry, detection techniques, response actions, EDR vs antivirus |
| [siem.md](02-siem.md) | SIEM | Log sources, why isolated logs fail, parsing vs normalization, correlation, detection rules |
| [splunk.md](03-splunk.md) | Splunk | Forwarder, indexer, search head, uploading data, my first SPL searches |
| [elk.md](04-elk.md) | Elastic Stack (ELK) | Beats, Logstash, Elasticsearch, Kibana, KQL, visualizations, dashboards |
| [soar.md](05-soar.md) | SOAR | Orchestration vs automation, playbooks (phishing, CVE patching) |
| [pyramid-of-pain.md](06-pyramid-of-pain.md) | Pyramid of Pain | All six indicator levels, Fast Flux, Punycode, sandbox reports, fuzzy hashing, TTPs |

## How it all fits together

```
Logs from endpoints, network, email, identity
        │
        ▼
SIEM (Splunk / ELK)  ── collects, normalizes, correlates, alerts
        │
        ▼
SOAR  ── runs a playbook: enrich with threat intel, decide, act
        │
        ▼
EDR / Firewall / IAM  ── isolate host, block IP, disable user
        │
        ▼
Analyst  ── judgment calls, deeper investigation, tuning rules
```

## Quick reference

**Windows Event IDs I need to know**

| Event ID | Meaning |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4688 | New process created |
| 104 | Event log cleared |

**True vs false positive**

1. Investigate the alert and the events behind it
2. Decide: real threat (true positive) or false alarm (false positive)
3. False positive → tune the rule so it stops firing on harmless activity
4. True positive → investigate further, contact the asset owner, contain (isolate host, block IP)
