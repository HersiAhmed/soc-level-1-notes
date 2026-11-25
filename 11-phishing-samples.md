# Phishing Samples in Action

Analysis of six phishing samples, each adding a technique. Together they cover the full attacker playbook, from stealing a password to running code on the victim's machine. The method is the same every time: read the header, inspect the raw source instead of clicking, and check where links and files really go.

## The recurring red flags

| Red flag | Why it matters |
|---|---|
| Display name vs real address | The display name can say anything; the real sending address is what counts |
| Urgency | "Suspended", "Action Required", short expiry windows push you to act before thinking |
| Brand impersonation | Real logos and HTML borrow a trusted brand's credibility |
| Link vs destination | The visible text/button and the real `href` are often different |
| Odd recipients (BCC, wrong "To") | Signals a mass blast with a hidden recipient list |
| Typos / inconsistencies | Still useful, but unreliable now that AI writes clean phishing |

---

## Sample 1: PayPal receipt — spoofing + URL shortener

A fake PayPal gift-card receipt. The only interactive element is a **Cancel the order** button, designed to make you panic at a charge you didn't make.

- **From:** displays `service@paypal.com`, actually from `gibberish@sultanbogor.com`
- The button links to a **shortened URL**, hiding the real destination
- **Tool:** [WhereGoes](https://wheregoes.com/) follows a shortened link's redirects and shows the final destination without visiting it

**Lesson:** never click a shortened link blind; expand it first.

---

## Sample 2: Package tracking — tracking pixel + link manipulation

A fake "Track your package" email. Yahoo blocked its images and links automatically.

- **From:** "Distribution Center" `<contact@beginpro.club>` — display name doesn't match the domain
- In the raw source, the **display text** said "Track your package" but the **href** pointed to `devret.xyz`, an unrelated domain
- Contained a **tracking pixel**: `<img src="http://devret.xyz/Creatives/tracking.png">`

**Tracking pixel (web bug):** a tiny invisible image. When your mail app loads it, your machine requests it from the attacker's server, confirming your address is active and reporting that you opened the email, plus your IP. **This is why email clients block images by default** — if the image never loads, the attacker never gets the ping.

Defanging the destination: `hxxp[://]devret[.]xyz/...`, root domain `devret[.]xyz`.

---

## Sample 3: Multi-stage credential harvesting

The most elaborate: a fake Citrix "you have a new fax" email leading through a redirect chain.

**The chain:** Citrix email → fake **OneDrive** page (`app.poptin.in`) → fake **Adobe** page (`bdkmotorsport.com/wp-duua`, a hacked WordPress site) → fake **Outlook login**.

- Each stage layers a different trusted brand to build false confidence
- The final page is a **credential-harvesting portal**
- Key detail: **even correct credentials return a generic "Invalid Credentials" error.** The page isn't authenticating anyone; it just captures whatever is typed and sends it to the attacker, then shows an error so the victim blames a typo

**Lesson:** a page asking you to "sign in with your email provider to view a document" is a credential trap. Also, don't rely on grammar/typos — AI-generated phishing is clean now; rely on structural tells (wrong URLs, mismatched brands).

---

## Sample 4: Netflix billing — malicious link inside a PDF

Fake "Netfllx ID Suspended" (note the misspelling) billing alert.

- **From:** "Netllx billing" `<z99@musacombi.online>` — misspelled display name and an unrelated domain
- Recipient was **BCC'd**, a sign of a mass send
- The malicious link lived **inside a PDF attachment** ("Update Payment Account"), not in the email body — this helps bypass filters that only scan body text
- Included one **real** `help.netflix.com` link to seem legit, and an **odd phone number format** (a vishing angle)

**Lesson:** an attachment isn't safer than a link; attackers hide links inside PDFs to slip past filters. One genuine link next to a fake one doesn't make the email safe.

---

## Sample 5: Apple receipt — blank body + macro-capable attachment + BCC

Fake "Action Required: your recent purchase" App Store receipt.

- **From:** "Apple Support" `<...@sumpremed.com>`; **To:** a fake `...apple.iud456.com` address with a typo ("payament"); the real victim was **BCC'd**
- The email **body was completely blank** — suspicious on its own
- The attachment was a **`.dot`** file (a Word template), the wrong format for a receipt and capable of carrying macros
- Clicking the embedded image fired a redirect: `t.tumblr.com/redirect?z=https://apps.ios.games.mansiiea.com/...` — abusing Tumblr's redirector, with "apps"/"ios" sprinkled in to look Apple-ish

**BCC (Blind Carbon Copy):** sends a hidden copy that other recipients can't see. Attackers put a fake address in "To" and BCC the real victims, blasting many people while hiding the list. If you receive an email whose "To" isn't you, you were BCC'd — a red flag for a legit-looking billing email.

**Office file macro risk:**

| Extension | Type | Macros |
|---|---|---|
| `.docx` / `.xlsx` | Normal document | No |
| `.docm` / `.xlsm` | Macro-enabled | Yes |
| `.dot` / `.dotm` | Word template | Yes |

---

## Sample 6: DHL shipping — Excel attachment that runs an executable

The most dangerous: this one tries to run code, not just steal a password.

- **From:** "DHL Express" `<info@glamcarcompany.de>` — a car-company domain
- The **`.xlsx`** attachment had glaring geographic inconsistencies: a **German** sender, an invoice addressed to **India**, and document text in **Mandarin**. Mismatched countries in one document = reused/stitched template
- A red banner said **"TO TRANSLATE LANGUAGE KINDLY CLICK ENABLE EDITING"** — the classic lure to enable macros
- The macro tried to download and run **`regasms.exe`** from `C:\Users\admin\AppData\Roaming\` (a writable folder malware favors; the name masquerades as Microsoft's real RegAsm.exe). It crashed here, but the intent was code execution

**If it had run**, the attacker could establish persistence (backdoor/scheduled task), exfiltrate data and credentials, or deploy ransomware — the same kill-chain phases from the frameworks rooms.

**Lesson:** this is malware phishing, not credential phishing. An Office attachment prompting "Enable Editing/Content" is one of the most common malware delivery methods. Never enable content; detonate the file in a sandbox instead.

---

## Two kinds of phishing payoff

| | Goal | Example here |
|---|---|---|
| **Credential phishing** | Steal your password | Samples 1–3 (fake login pages) |
| **Malware phishing** | Run code on your machine | Samples 4–6 (malicious attachments) |

## My analysis habit

1. **Header:** display name vs real address, Reply-To, the real "To" vs BCC
2. **Body:** urgency, brand impersonation, inconsistencies
3. **Links:** read the real `href` in the source; expand shorteners with WhereGoes; never click
4. **Attachments:** suspicious types (`.dot`, `.docm`, `.xlsm`), macro lures; detonate in a sandbox
5. **Defang** every indicator before writing it down: `hxxp[://]bad[.]domain`
