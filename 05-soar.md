# Security Orchestration, Automation, and Response (SOAR)

## What a SOC does

| Capability | Tools | Example |
|---|---|---|
| Monitoring and detection | SIEM | Many failed logins, login from an unknown location |
| Recovery and remediation | EDR, firewall, IAM | Isolate an endpoint, block an IP, disable a user |
| Threat intelligence | TI feeds | Block a domain flagged as malicious |
| Communication | Ticketing | Ask IT to verify a new patch |

## The problems SOAR solves

| Challenge | What it means |
|---|---|
| Alert fatigue | Too many alerts, many of them false positives; real threats get missed |
| Disconnected tools | Firewall, EDR, and SIEM don't talk to each other |
| Manual processes | Steps live in senior analysts' heads ("tribal knowledge"), not in documentation |
| Talent shortage | Not enough analysts for the volume of alerts |

## The three capabilities

The easiest way I've found to tell the first two apart: **orchestration is the plan, automation is running the plan without a human.**

**Orchestration** connects tools from different vendors into one interface and defines **playbooks**: step-by-step workflows for a type of alert.

**Automation** runs the playbook with no manual clicks.

**Response** takes action across tools from one place: block the IP on the firewall, disable the user in IAM, open a ticket.

### Example: VPN brute force alert

Without SOAR, the analyst jumps between the SIEM, a threat intel platform, IAM, and the ticketing system.

With SOAR, the playbook runs automatically:

1. Receive the alert from the SIEM
2. Query the SIEM: does this user normally log in from this IP?
3. Check threat intel: what's the IP's reputation?
4. Query the SIEM: any successful logins?
5. If malicious → disable the user in IAM, block the IP
6. Open a ticket with all the details

Playbooks **branch**. If the user always uses that IP and there were only a few failures, it's probably a forgotten password, so the playbook stops early.

## Example playbooks

### Phishing

```
Suspicious email received
        │
   Create ticket
        │
 URL or attachment? ── No ──► Notify user, close
        │ Yes
        ├── URL ──────► Check reputation in threat intel
        └── Attachment ► Check hash / detonate in a sandbox
        │
   Malicious? ── No ──► Notify user, close
        │ Yes
   Remediate: remove from all inboxes, block sender/URL
        │
   Analyst review, close ticket
```

### CVE patching

A **CVE** is a publicly disclosed vulnerability with an ID (like a public recall notice for software). New ones come out constantly, which creates a backlog if handled by hand.

```
New CVE published
        │
 Analyze details: what's affected?
        │
 In our environment? ── No ──► Close
        │ Yes
 Assess risk / severity
        │
 Create patching ticket
        │
 Test the patch in a test environment
        │
 Deploy to production
```

**Where new CVEs come from:** public feeds such as the NVD, CVE.org, CISA's Known Exploited Vulnerabilities list, and vendor advisories (e.g. Microsoft's Patch Tuesday), usually pulled in through APIs.

**How SOAR knows if we're affected:** vulnerability scanners (e.g. Nessus, Qualys) and the asset inventory.

**Why test first:** a bad patch can break production systems.

## Does SOAR replace analysts?

No. Both playbooks still have an analyst at key decision points. SOAR can't make judgment calls ("is shutting this server down worth the business impact?"), complex investigations still need a person, and **analysts are the ones who write the playbooks**.

SOAR takes the repetitive work off the team so analysts can focus on the parts that need a human.
