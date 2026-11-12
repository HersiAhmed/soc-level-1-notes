# Phishing and Email Analysis

## How email travels

| Protocol | Job | Postal analogy | Common ports |
|---|---|---|---|
| **SMTP** | Sends email | The mail truck | 25 (server to server), 587 (client sending) |
| **POP3** | Downloads to one device | Taking your mail home | 110, 995 (encrypted) |
| **IMAP** | Keeps mail on the server, synced across devices | Mail stays at the post office | 143, 993 (encrypted) |

```
Sender → (SMTP) → sender's mail server → DNS (MX record) → (SMTP) → recipient's server → (POP3/IMAP) → recipient
```

The **MX record** in DNS tells the sending server where to deliver mail for a domain. IMAP is the modern default.

## Email headers (the envelope)

Basic fields: **From**, **To**, **Reply-To**, **Subject**, **Date**. In Thunderbird, **View → Message Source** (Ctrl+U) shows the full raw email.

**Red flags in headers:**
- **Display name vs real address:** "ADT Security Services" sending from `newsletters@ant.anki-tech.com`
- **Reply-To** set to a different address than From, so replies go to the attacker

**`Received:` lines** are added by every mail server the email passes through. Each new server adds its line at the top, so **read them bottom to top** to trace the real path. The bottom one is closest to the real sender and gives the **originating IP**, which you can check on VirusTotal or AbuseIPDB. These are much harder to fake than the From field.

## Email body

| Plain text | HTML |
|---|---|
| Just words | Images, buttons, links, styling |
| Hard to hide anything | Easy to hide where a link really goes |

In the raw HTML:

```html
<a href="https://ant.anki-tech.com/ga/click/2-952622232-...">
   <img src="http://blueoceanfresh.com/img/15ef9a6...">
</a>
```

- `href` = where the link **actually** goes. The visible text or image can say anything.
- Remote `img` images can work as **web bugs**: loading them tells the sender you opened the email, plus your IP. That's why email clients block them by default.
- `=3D` is quoted-printable encoding for `=`.

## Attachments

Attachments are stored as text inside the email:

| Header | Meaning |
|---|---|
| `Content-Type: application/pdf` | File type |
| `Content-Disposition: attachment; filename="..."` | It's an attachment, and its name |
| `Content-Transfer-Encoding: base64` | How the file was turned into text |

The base64 block can be decoded back into the original file with CyberChef ("From Base64"). PDFs in base64 almost always start with `JVBERi0` (`%PDF-`). In real work, suspicious files get opened in a sandbox, never on your own machine.

## Types of malicious messages

| Type | Target |
|---|---|
| Spam / malspam | Everyone, in bulk (malspam carries malware) |
| Phishing | Many people, posing as a trusted brand |
| Spear phishing | One specific person or company |
| Whaling | Executives (CEO, CFO) |
| Smishing | SMS / text messages |
| Vishing | Voice / phone calls (including "call this number" phishing) |

## Common red flags

- Spoofed or lookalike sender (`microsof.com`, Punycode)
- Urgency ("Last notice: your account will be locked in 48 hours")
- Brand impersonation
- Generic greetings ("Dear Customer")
- Hidden or shortened links
- Double extensions (`invoice.pdf.exe`: the real extension is the last one)
- Odd subjects, e.g. starting with "Re:" when you never wrote, or random characters to dodge spam filters
- Spelling mistakes used to be a giveaway, but AI-written phishing is now often clean

A "Trusted Sender" label or passing authentication can just mean the email really came from the attacker's own domain.

## Defanging

Making links and IPs unclickable before writing them in tickets or reports:

| Original | Defanged |
|---|---|
| `http://www.suspiciousdomain.com` | `hxxp[://]www[.]suspiciousdomain[.]com` |
| `192.168.1.1` | `192[.]168[.]1[.]1` |

CyberChef has "Defang URL" and "Defang IP Addresses" recipes.

## My analysis checklist

1. **Headers:** display name vs address, Reply-To, `Received:` lines and originating IP
2. **Body:** urgency, generic greeting, brand impersonation
3. **Links:** check every `href` in the source (never click)
4. **Attachments:** Content-Type, filename, double extensions
5. **Defang** every indicator before writing it down
