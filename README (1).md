# Phishing Email Investigation Reports

> Three phishing email samples analyzed using OSINT tools: MXToolbox, AbuseIPDB, VirusTotal, URLScan.io, and WHOIS lookups.

---

## Table of Contents
- [Case 01 — Fake SBI KYC Alert](#case-01--fake-sbi-kyc-alert)
- [Case 02 — Fake Amazon Order Cancellation](#case-02--fake-amazon-order-cancellation)
- [Case 03 — Fake Income Tax Refund](#case-03--fake-income-tax-refund)
- [Case 04 — Fake WhatsApp Subscription Expiry](#case-04--fake-whatsapp-subscription-expiry)
- [Case 05 — Fake HDFC Credit Card Alert](#case-05--fake-hdfc-credit-card-alert)

---

# Case 01 — Fake SBI KYC Alert

## Raw Sample: `01-fake-sbi-kyc-alert.eml`

```eml
From: SBI Customer Care <alert-noreply@sbi-secure-verify.in>
To: customer@gmail.com
Subject: URGENT: Your SBI Account Will Be Blocked in 24 Hours - KYC Verification Required
Date: Mon, 14 Apr 2026 09:23:15 +0530
Message-ID: <20260414092315.ABC123@sbi-secure-verify.in>
MIME-Version: 1.0
Content-Type: text/html; charset="UTF-8"
Return-Path: <bounce@cheap-mailer.xyz>
Received: from cheap-mailer.xyz (185.234.72.19) by mx.google.com with SMTP;
        Mon, 14 Apr 2026 09:23:16 +0530
X-Mailer: PHPMailer 5.2.9
Authentication-Results: mx.google.com;
        spf=fail (sender IP is 185.234.72.19)
        dkim=none
        dmarc=fail
```

---

# Investigation: URGENT: Your SBI Account Will Be Blocked in 24 Hours - KYC Verification Required

## Quick Summary

| Field | Value |
|---|---|
| Sample file | `01-fake-sbi-kyc-alert.eml` |
| Date received | 2026-04-14 |
| Subject | URGENT: Your SBI Account Will Be Blocked in 24 Hours - KYC Verification Required |
| Sender (display name) | SBI Customer Care |
| Sender (actual email) | alert-noreply@sbi-secure-verify.in |
| Sender IP | 185.234.72.19 |
| Attack type | Credential Harvesting |
| Brand impersonated | State Bank of India (SBI) |
| Verdict | **PHISHING** |

---

## Step 1 — Initial Triage

What I noticed in the first 30 seconds:

- **Urgency language?** YES — *"Your SBI Account Will Be Blocked in 24 Hours"* and *"BLOCKED within 24 hours if KYC verification is not completed"*
- **Threatening consequences?** YES — account block/service disruption threatened
- **Suspicious sender address?** YES — sent from `sbi-secure-verify.in`, which is not SBI's real domain (`sbi.co.in`)
- **Links present?** YES — 1 phishing link to `https://sbi-secure-verify.in/kyc/verify.php?ref=KYC2026041401`
- **Attachments?** None

---

## Step 2 — Header Analysis

Key headers from the `.eml` file:

- **From:** SBI Customer Care `<alert-noreply@sbi-secure-verify.in>`
- **Return-Path:** `<bounce@cheap-mailer.xyz>`
- **Received (first hop):** from cheap-mailer.xyz (185.234.72.19) by mx.google.com
- **Sender IP:** 185.234.72.19
- **X-Mailer:** PHPMailer 5.2.9

Authentication results:

- **SPF:** FAIL (sender IP is 185.234.72.19)
- **DKIM:** NONE
- **DMARC:** FAIL

**Red flags found:**

- Return-Path doesn't match From address? **YES** — Return-Path is `bounce@cheap-mailer.xyz`, From is `@sbi-secure-verify.in`
- Sender IP in a different country than claimed? **YES** — IP geolocates to Germany (Frankfurt); SBI is an Indian bank
- X-Mailer or unusual headers? **YES** — `PHPMailer 5.2.9` (outdated bulk mailer), suspicious `cheap-mailer.xyz` relay

---

## Step 3 — Sender Investigation

- **Sender domain:** `sbi-secure-verify.in`
- **WHOIS registration date:** No WHOIS data found — domain either non-existent or recently created with hidden registration
- **Registrar:** Unknown (WHOIS lookup returned no data for `alert-noreplysbi-secure-verify.in`)
- **AbuseIPDB reports:** 0 reports on IP 185.234.72.19 — freshly provisioned, not yet flagged
- **VirusTotal domain check:** 0/91 vendors flagged (newly registered domain, not yet in threat feeds)

**Analysis:** `sbi-secure-verify.in` is a look-alike domain designed to impersonate SBI. The real SBI domain is `sbi.co.in`. The Return-Path pointing to `cheap-mailer.xyz` and the sender IP geolocating to Germany (while impersonating an Indian bank) are strong indicators of fraud.

---

## Step 4 — URL Analysis

URLs found in the email body (defanged):

- `hxxps://sbi-secure-verify[.]in/kyc/verify[.]php?ref=KYC2026041401&acc=4521`

For the phishing URL:

- **URLScan.io result:** HTTP 400 Error — DNS resolution failed; domain `sbi-secure-verify.in` could not be resolved to a valid IPv4/IPv6 address (domain is offline/taken down)
- **VirusTotal:** 0/91 vendors flagged — URL not yet in threat intelligence feeds
- **Does domain match the real brand?** NO — real SBI domain is `sbi.co.in`; this uses `sbi-secure-verify.in`
- **Redirect chain:** Domain non-resolvable at time of analysis

---

## Step 5 — Attachment Analysis

No attachments present.

---

## Step 6 — Extracted IOCs

| Type | Value | Context |
|---|---|---|
| Email | alert-noreply@sbi-secure-verify.in | Sender address |
| Email | bounce@cheap-mailer.xyz | Return-Path (actual mailer) |
| IP | 185.234.72.19 | Sending server (Frankfurt, Germany — DeinServerHost / AS213250) |
| Domain | sbi-secure-verify.in | Phishing domain impersonating SBI |
| Domain | cheap-mailer.xyz | Bulk mail relay used to send phishing email |
| URL | hxxps://sbi-secure-verify[.]in/kyc/verify[.]php?ref=KYC2026041401&acc=4521 | Phishing link in email body |

---

## Step 7 — Attack Classification

- **Type:** Credential Harvesting
- **Target:** General public (SBI retail banking customers)
- **Sophistication:** Medium — uses branded HTML template, fake RBI regulatory urgency, and a convincing look-alike domain
- **MITRE ATT&CK:**
  - T1566.002 — Phishing: Spearphishing Link
  - T1598.003 — Phishing for Information: Spearphishing Link

---

## Step 8 — Recommended Actions

- [x] Block sender domain `sbi-secure-verify.in` at email gateway
- [x] Block sender IP `185.234.72.19` at firewall
- [x] Block relay domain `cheap-mailer.xyz` at email gateway
- [x] Add phishing URL to proxy blocklist
- [x] Submit to PhishTank for community reporting
- [x] Alert users who may have received similar emails
- [x] If any user clicked: reset credentials, check for data exposure

---

## Screenshots — Case 01

### AbuseIPDB (185.234.72.19)
![AbuseIPDB results for 185.234.72.19](AbuseIPDB_results.png)

> IP hosted at DeinServerHost (AS213250), Frankfurt am Main, Germany. Reported 0 times — freshly provisioned infrastructure.

---

### MXToolbox (sbi-secure-verify.in)
![MXToolbox results for sbi-secure-verify.in](MXToolbox_results.png)

> 10 critical problems detected: no DNS records, no MX records, no SPF, no DMARC. The domain has zero legitimate email infrastructure — a clear sign of a throwaway phishing domain.

---

### URLScan.io (sbi-secure-verify.in — live scan)
![URLScan.io HTTP 400 DNS error](urlscan_io.png)

> Domain could not be resolved at time of scanning — the phishing site was already offline or never properly hosted.

---

### VirusTotal (phishing URL)
![VirusTotal results 0/91](virus_total_results.png)

> 0/91 security vendors flagged the URL. Newly registered domains frequently evade detection initially — the 0/91 score does **not** mean the URL is safe.

---

### WHOIS (alert-noreplysbi-secure-verify.in)
![WHOIS lookup results](whois.png)

> No WHOIS data found. The domain either does not exist as a registered entity or uses privacy shielding — typical of short-lived phishing infrastructure.

---
---

# Case 02 — Fake Amazon Order Cancellation

## Raw Sample: `02-fake-amazon-order-cancellation.eml`

```eml
From: Amazon India <order-update@amazon-notifications-in.com>
To: customer@gmail.com
Subject: Your Amazon Order #402-8834921-7721534 Has Been Cancelled - Action Required
Date: Tue, 15 Apr 2026 14:45:30 +0530
Message-ID: <20260415144530.DEF456@amazon-notifications-in.com>
MIME-Version: 1.0
Content-Type: text/html; charset="UTF-8"
Return-Path: <noreply@amazon-notifications-in.com>
Received: from mail.amazon-notifications-in.com (91.215.41.33) by mx.google.com with SMTP;
        Tue, 15 Apr 2026 14:45:31 +0530
Authentication-Results: mx.google.com;
        spf=fail (sender IP is 91.215.41.33)
        dkim=none
        dmarc=fail
```

---

# Investigation: Your Amazon Order #402-8834921-7721534 Has Been Cancelled - Action Required

## Quick Summary

| Field | Value |
|---|---|
| Sample file | `02-fake-amazon-order-cancellation.eml` |
| Date received | 2026-04-15 |
| Subject | Your Amazon Order #402-8834921-7721534 Has Been Cancelled - Action Required |
| Sender (display name) | Amazon India |
| Sender (actual email) | order-update@amazon-notifications-in.com |
| Sender IP | 91.215.41.33 |
| Attack type | Credential Harvesting |
| Brand impersonated | Amazon India |
| Verdict | **PHISHING** |

---

## Step 1 — Initial Triage

What I noticed in the first 30 seconds:

- **Urgency language?** YES — *"Action Required"* in the subject line; email implies payment compromise
- **Threatening consequences?** YES — order cancellation, payment method compromise implied
- **Suspicious sender address?** YES — sent from `amazon-notifications-in.com`; real Amazon India uses `amazon.in` or `amazon.com`
- **Links present?** YES — 1 phishing link to `https://amazon-notifications-in.com/verify-payment?order=4028834921`
- **Attachments?** None

---

## Step 2 — Header Analysis

Key headers from the `.eml` file:

- **From:** Amazon India `<order-update@amazon-notifications-in.com>`
- **Return-Path:** `<noreply@amazon-notifications-in.com>`
- **Received (first hop):** from mail.amazon-notifications-in.com (91.215.41.33) by mx.google.com
- **Sender IP:** 91.215.41.33

Authentication results:

- **SPF:** FAIL (sender IP is 91.215.41.33)
- **DKIM:** NONE
- **DMARC:** FAIL

**Red flags found:**

- Return-Path doesn't match From address? **NO** — both use `amazon-notifications-in.com`, but that domain itself is fake
- Sender IP in a different country than claimed? **YES** — IP geolocates to Russia (Rostov-na-Donu); Amazon India would send from Amazon's own infrastructure
- X-Mailer or unusual headers? **None visible**, but SPF/DKIM/DMARC all fail

---

## Step 3 — Sender Investigation

- **Sender domain:** `amazon-notifications-in.com`
- **WHOIS registration date:** IP WHOIS shows RIPE NCC allocation (91.0.0.0/8 range, RegDate 2005-06-30) — but this is the regional registry block, not the actual domain registration date
- **Registrar:** RIPE NCC (IP allocation); domain registrar unknown
- **AbuseIPDB reports:** 0 reports on IP 91.215.41.33 — not yet flagged; hosted at DDOS-GUARD LTD, Rostov-na-Donu, Russia (AS57724)
- **VirusTotal domain check:** Not flagged at time of analysis

**Analysis:** `amazon-notifications-in.com` is a look-alike domain designed to spoof Amazon India. The real Amazon India notification domain is `amazon.in`. The sender IP routing through DDOS-GUARD LTD in Russia is a major red flag for an email claiming to be from an Indian e-commerce platform.

---

## Step 4 — URL Analysis

URLs found in the email body (defanged):

- `hxxps://amazon-notifications-in[.]com/verify-payment?order=4028834921&ref=AMZIN2026`

For the phishing URL:

- **URLScan.io result:** HTTP 400 Error — DNS resolution failed; domain `amazon-notifications-in.com` could not be resolved to a valid IPv4/IPv6 address
- **VirusTotal:** 0/91 vendors flagged — newly registered domain not yet in threat feeds
- **Does domain match the real brand?** NO — Amazon India's real domain is `amazon.in`
- **Redirect chain:** Domain non-resolvable at time of analysis

---

## Step 5 — Attachment Analysis

No attachments present.

---

## Step 6 — Extracted IOCs

| Type | Value | Context |
|---|---|---|
| Email | order-update@amazon-notifications-in.com | Sender address |
| IP | 91.215.41.33 | Sending server (Rostov-na-Donu, Russia — DDOS-GUARD LTD / AS57724) |
| Domain | amazon-notifications-in.com | Phishing domain impersonating Amazon India |
| URL | hxxps://amazon-notifications-in[.]com/verify-payment?order=4028834921&ref=AMZIN2026 | Phishing link in email body |

---

## Step 7 — Attack Classification

- **Type:** Credential Harvesting (targeting Amazon account credentials and/or payment card details)
- **Target:** General public (Amazon India customers)
- **Sophistication:** Medium — realistic order details (plausible order number, product name, price in INR), branded HTML mimicking Amazon's design
- **MITRE ATT&CK:**
  - T1566.002 — Phishing: Spearphishing Link
  - T1598.003 — Phishing for Information: Spearphishing Link

---

## Step 8 — Recommended Actions

- [x] Block sender domain `amazon-notifications-in.com` at email gateway
- [x] Block sender IP `91.215.41.33` at firewall
- [x] Add phishing URL to proxy blocklist
- [x] Submit to PhishTank for community reporting
- [x] Alert users who may have received similar emails
- [x] If any user clicked: reset Amazon credentials, check for unauthorized payment method access, notify bank if card details were entered

---

## Screenshots — Case 02

### AbuseIPDB (91.215.41.33)
![AbuseIPDB results for 91.215.41.33](AbuseIPDB.png)

> IP hosted at DDOS-GUARD LTD (AS57724), Rostov-na-Donu, Russian Federation. 0 abuse reports — freshly provisioned or recently rotated infrastructure. Use of a Russian DDoS-protection/hosting service is a significant red flag for an email claiming to be from Amazon India.

---

### MXToolbox (amazon-notifications-in.com)
![MXToolbox results for amazon-notifications-in.com](Screenshot_2026-04-22_130055.png)

> 10 critical problems: no HTTP resolution, no DMARC, no MX records, no SPF. The domain has no legitimate email infrastructure configured — consistent with a throwaway phishing domain.

---

### URLScan.io (amazon-notifications-in.com — live scan)
![URLScan.io HTTP 400 DNS error](urlscan.png)

> DNS resolution failed — the domain `amazon-notifications-in.com` could not be resolved at time of scanning. The phishing site was offline at time of analysis.

---

### VirusTotal (phishing URL)
![VirusTotal 0/91](virustotal.png)

> 0/91 vendors flagged the URL `https://amazon-notifications-in.com/verify-payment?order=4028834921`. As with Case 01, a clean VirusTotal result on a newly registered look-alike domain is **not** an indicator of safety.

---

### WHOIS (IP 91.215.41.33)
![WHOIS IP lookup](who_is.png)

> IP WHOIS returns RIPE NCC block allocation data (91.0.0.0/8, RegDate 2005-06-30, NL). The actual hosting assignment is DDOS-GUARD LTD in Russia per AbuseIPDB — RIPE data reflects the regional registry, not the current operator.

---
---

# Case 03 — Fake Income Tax Refund

## Raw Sample: `03-fake-income-tax-refund.eml`

```eml
From: Income Tax Department <refund-notice@incometax-gov-refund.in>
To: customer@gmail.com
Subject: Income Tax Refund of Rs. 23,847 Approved - Verify Bank Details
Date: Wed, 16 Apr 2026 10:12:00 +0530
Message-ID: <20260416101200.GHI789@incometax-gov-refund.in>
MIME-Version: 1.0
Content-Type: text/html; charset="UTF-8"
Return-Path: <mailer@incometax-gov-refund.in>
Received: from incometax-gov-refund.in (103.74.19.88) by mx.google.com with SMTP;
        Wed, 16 Apr 2026 10:12:01 +0530
Authentication-Results: mx.google.com;
        spf=fail (sender IP is 103.74.19.88)
        dkim=none
        dmarc=fail
```

---

# Investigation: Income Tax Refund of Rs. 23,847 Approved - Verify Bank Details

## Quick Summary

| Field | Value |
|---|---|
| Sample file | `03-fake-income-tax-refund.eml` |
| Date received | 2026-04-16 |
| Subject | Income Tax Refund of Rs. 23,847 Approved - Verify Bank Details |
| Sender (display name) | Income Tax Department |
| Sender (actual email) | refund-notice@incometax-gov-refund.in |
| Sender IP | 103.74.19.88 |
| Attack type | Credential Harvesting / Financial Fraud |
| Brand impersonated | Government of India — Income Tax Department |
| Verdict | **PHISHING** |

---

## Step 1 — Initial Triage

What I noticed in the first 30 seconds:

- **Urgency language?** YES — *"please verify your bank account details within 48 hours"*
- **Threatening consequences?** YES — implied loss of tax refund (Rs. 23,847) if action not taken
- **Suspicious sender address?** YES — sent from `incometax-gov-refund.in`; the real Income Tax Department uses `incometax.gov.in`
- **Links present?** YES — 1 phishing link to `https://incometax-gov-refund.in/refund/verify-bank?pan=BNZPS2841K`
- **Attachments?** None

---

## Step 2 — Header Analysis

Key headers from the `.eml` file:

- **From:** Income Tax Department `<refund-notice@incometax-gov-refund.in>`
- **Return-Path:** `<mailer@incometax-gov-refund.in>`
- **Received (first hop):** from incometax-gov-refund.in (103.74.19.88) by mx.google.com
- **Sender IP:** 103.74.19.88

Authentication results:

- **SPF:** FAIL (sender IP is 103.74.19.88)
- **DKIM:** NONE
- **DMARC:** FAIL

**Red flags found:**

- Return-Path doesn't match From address? **NO** — both use `incometax-gov-refund.in`, but this domain is fraudulent
- Sender IP in a different country than claimed? **NO** — IP geolocates to India (Karjat, Maharashtra — Zess Networks), which makes this attack more convincing than Cases 01 and 02
- X-Mailer or unusual headers? **None visible**, but SPF/DKIM/DMARC all fail
- Domain impersonation? **YES** — `incometax-gov-refund.in` mimics `incometax.gov.in` by inserting `-gov-refund` as a subdomain-style string in the domain name

---

## Step 3 — Sender Investigation

- **Sender domain:** `incometax-gov-refund.in`
- **WHOIS registration date:** WHOIS data not available via standard lookup — domain lacks public registration records
- **Registrar:** Unknown
- **AbuseIPDB reports:** 0 reports on IP 103.74.19.88 — not yet flagged; hosted at Zess Networks Private Limited, Karjat, Maharashtra, India (AS133720, Fixed Line ISP)
- **VirusTotal domain check:** Not flagged at time of analysis

**Analysis:** `incometax-gov-refund.in` is a carefully crafted look-alike designed to impersonate the Government of India's Income Tax portal (`incometax.gov.in`). The use of Indian hosting (Zess Networks, Maharashtra) and a `.in` TLD makes this attack more geographically convincing than the previous two cases. The real IT Department's refund process goes through `incometax.gov.in` and SBI's refund banker — never through third-party bank verification links.

---

## Step 4 — URL Analysis

URLs found in the email body (defanged):

- `hxxps://incometax-gov-refund[.]in/refund/verify-bank?pan=BNZPS2841K&amt=23847`

For the phishing URL:

- **URLScan.io result:** HTTP 400 Error — DNS resolution failed; domain `incometax-gov-refund.in` could not be resolved to a valid IPv4/IPv6 address
- **VirusTotal:** 0/91 vendors flagged — domain not yet in threat intelligence feeds
- **Does domain match the real brand?** NO — real Income Tax Department portal is `incometax.gov.in`; this uses `incometax-gov-refund.in` (a `.in` domain, not `.gov.in`)
- **Redirect chain:** Domain non-resolvable at time of analysis
- **Note:** The URL embeds a partial PAN number (`BNZPS2841K`) — this is likely a social engineering tactic to make the link appear personalized and legitimate

---

## Step 5 — Attachment Analysis

No attachments present.

---

## Step 6 — Extracted IOCs

| Type | Value | Context |
|---|---|---|
| Email | refund-notice@incometax-gov-refund.in | Sender address |
| Email | mailer@incometax-gov-refund.in | Return-Path |
| IP | 103.74.19.88 | Sending server (Karjat, Maharashtra, India — Zess Networks / AS133720) |
| Domain | incometax-gov-refund.in | Phishing domain impersonating Income Tax Department |
| URL | hxxps://incometax-gov-refund[.]in/refund/verify-bank?pan=BNZPS2841K&amt=23847 | Phishing link in email body |

---

## Step 7 — Attack Classification

- **Type:** Credential Harvesting / Financial Fraud (targeting bank account details, potentially full banking credentials)
- **Target:** General public (Indian taxpayers)
- **Sophistication:** High — uses a `.gov.in`-style look-alike TLD structure, references real tax concepts (Assessment Year 2025-26, PAN, CPC Bengaluru, refund helpline number), embeds a personalised PAN fragment in the URL, and sends from Indian IP infrastructure. This is the most convincing of the three samples.
- **MITRE ATT&CK:**
  - T1566.002 — Phishing: Spearphishing Link
  - T1598.003 — Phishing for Information: Spearphishing Link
  - T1056 — Input Capture (likely objective of the landing page — capturing bank account/card details)

---

## Step 8 — Recommended Actions

- [x] Block sender domain `incometax-gov-refund.in` at email gateway
- [x] Block sender IP `103.74.19.88` at firewall
- [x] Add phishing URL to proxy blocklist
- [x] Submit to PhishTank and report to CERT-In (cert-in.org.in) and the Income Tax Department's official cybercrime reporting channel
- [x] Alert users who may have received similar emails — emphasise that the real IT Department portal is `incometax.gov.in` and will never ask for bank details via email link
- [x] If any user clicked and entered bank details: contact bank immediately to freeze account, file complaint with cybercrime.gov.in

---

## Screenshots — Case 03

### AbuseIPDB (103.74.19.88)
![AbuseIPDB results for 103.74.19.88](abuseIPDB.png)

> IP hosted at Zess Networks Private Limited (AS133720), Karjat, Maharashtra, India. 0 abuse reports — clean record, but this is a Fixed Line ISP IP being used to send phishing email, which is suspicious. Domestic Indian hosting makes this attack harder to detect via geographic anomaly.

---

### MXToolbox (incometax-gov-refund.in)
![MXToolbox results for incometax-gov-refund.in](mxtoolbox.png)

> 10 critical problems: no DNS, no DMARC, no MX, no SPF. Identical failure pattern to Cases 01 and 02 — the domain has no legitimate email infrastructure despite appearing to be an official government domain.

---

### URLScan.io (incometax-gov-refund.in — live scan)
![URLScan.io HTTP 400 DNS error](urlscan.png)

> DNS resolution failed — `incometax-gov-refund.in` could not be resolved. The phishing site was offline or not properly hosted at time of scanning.

---

### VirusTotal (phishing URL)
![VirusTotal 0/91](virustotal.png)

> 0/91 vendors flagged the URL `https://incometax-gov-refund.in/refund/verify-bank?pan=BNZPS2841K`. As with the previous cases, a clean VirusTotal result on a newly created look-alike domain must not be interpreted as safe.

---

### WHOIS (IP 103.74.19.88)
![WHOIS IP lookup](whois.png)

> IP WHOIS returns APNIC block allocation (103.0.0.0/8, RegDate 2011-01-09, Australia). Actual operator per AbuseIPDB is Zess Networks Private Limited in Maharashtra, India — the APNIC data reflects the regional registry, not the current network operator.

---
---

# Case 04 — Fake WhatsApp Subscription Expiry

## Raw Sample: `04-fake-whatsapp-expiry.eml`

```eml
From: WhatsApp Team <support@whatsapp-renew-service.com>
To: customer@gmail.com
Subject: Your WhatsApp Subscription Has Expired! Renew Now to Avoid Deletion
Date: Thu, 17 Apr 2026 08:30:00 +0530
Message-ID: <20260417083000.JKL012@whatsapp-renew-service.com>
MIME-Version: 1.0
Content-Type: text/html; charset="UTF-8"
Return-Path: <bounce@whatsapp-renew-service.com>
Received: from whatsapp-renew-service.com (178.62.105.44) by mx.google.com with SMTP;
        Thu, 17 Apr 2026 08:30:01 +0530
Authentication-Results: mx.google.com;
        spf=fail (sender IP is 178.62.105.44)
        dkim=none
        dmarc=fail
```

---

# Investigation: Your WhatsApp Subscription Has Expired! Renew Now to Avoid Deletion

## Quick Summary

| Field | Value |
|---|---|
| Sample file | `04-fake-whatsapp-expiry.eml` |
| Date received | 2026-04-17 |
| Subject | Your WhatsApp Subscription Has Expired! Renew Now to Avoid Deletion |
| Sender (display name) | WhatsApp Team |
| Sender (actual email) | support@whatsapp-renew-service.com |
| Sender IP | 178.62.105.44 |
| Attack type | Credential Harvesting / Social Engineering (fake subscription) |
| Brand impersonated | WhatsApp (Meta Platforms) |
| Verdict | **PHISHING** |

---

## Step 1 — Initial Triage

What I noticed in the first 30 seconds:

- **Urgency language?** YES — *"Your WhatsApp Subscription Has EXPIRED"* and *"If you do not renew within 24 hours, your account and all chat history, photos, and videos will be permanently deleted"*
- **Threatening consequences?** YES — permanent deletion of account, chats, photos, videos threatened
- **Suspicious sender address?** YES — sent from `whatsapp-renew-service.com`; WhatsApp's real domain is `whatsapp.com`
- **Premise itself is fake?** YES — WhatsApp has always been free; there is **no subscription fee** to renew. The entire premise of the email is fabricated.
- **Links present?** YES — 1 phishing link to `https://whatsapp-renew-service.com/renew?phone=919876543210`
- **Attachments?** None

---

## Step 2 — Header Analysis

Key headers from the `.eml` file:

- **From:** WhatsApp Team `<support@whatsapp-renew-service.com>`
- **Return-Path:** `<bounce@whatsapp-renew-service.com>`
- **Received (first hop):** from whatsapp-renew-service.com (178.62.105.44) by mx.google.com
- **Sender IP:** 178.62.105.44

Authentication results:

- **SPF:** FAIL (sender IP is 178.62.105.44)
- **DKIM:** NONE
- **DMARC:** FAIL

**Red flags found:**

- Return-Path doesn't match From address? **NO** — both use `whatsapp-renew-service.com`, but that domain itself is fake
- Sender IP in a different country than claimed? **YES** — IP geolocates to London, UK (DigitalOcean); WhatsApp's legitimate infrastructure is run by Meta, not DigitalOcean
- X-Mailer or unusual headers? **None visible**, but SPF/DKIM/DMARC all fail

---

## Step 3 — Sender Investigation

- **Sender domain:** `whatsapp-renew-service.com`
- **WHOIS registration date:** Domain-level WHOIS not available; IP WHOIS shows RIPE NCC block (178.0.0.0/8, allocated 2009-01-30) — reflecting the regional registry allocation, not the current DigitalOcean tenant
- **Registrar:** Unknown for the domain; IP operator is DigitalOcean, LLC (AS14061)
- **AbuseIPDB reports:** 5 reports on IP 178.62.105.44 (confidence 0%) — some prior abuse history exists on this DigitalOcean address
- **VirusTotal IP check:** **2/94 vendors flag the IP as malicious** (Criminal IP, SOCRadar)
- **VirusTotal URL check:** 0/91 vendors flagged the phishing URL

**Analysis:** `whatsapp-renew-service.com` is a look-alike designed to trick users into believing WhatsApp has a paid subscription tier. This is a well-known scam pattern — WhatsApp has been **free since its acquisition by Facebook/Meta in 2014** and does not charge subscription fees. The use of a generic DigitalOcean London VPS that has attracted prior abuse reports confirms the malicious nature.

---

## Step 4 — URL Analysis

URLs found in the email body (defanged):

- `hxxps://whatsapp-renew-service[.]com/renew?phone=919876543210&token=abc123`

For the phishing URL:

- **URLScan.io result:** HTTP 400 Error — DNS resolution failed; domain `whatsapp-renew-service.com` could not be resolved to a valid IPv4/IPv6 address
- **VirusTotal:** 0/91 vendors flagged the URL
- **Does domain match the real brand?** NO — WhatsApp's real domain is `whatsapp.com`
- **Redirect chain:** Domain non-resolvable at time of analysis
- **Note:** The URL embeds an Indian phone number (`919876543210`) as a template parameter, suggesting the attacker personalises each link with the recipient's phone number to make the scam appear credible

---

## Step 5 — Attachment Analysis

No attachments present.

---

## Step 6 — Extracted IOCs

| Type | Value | Context |
|---|---|---|
| Email | support@whatsapp-renew-service.com | Sender address |
| Email | bounce@whatsapp-renew-service.com | Return-Path |
| IP | 178.62.105.44 | Sending server (London, UK — DigitalOcean / AS14061) |
| Domain | whatsapp-renew-service.com | Phishing domain impersonating WhatsApp |
| URL | hxxps://whatsapp-renew-service[.]com/renew?phone=919876543210&token=abc123 | Phishing link in email body |

---

## Step 7 — Attack Classification

- **Type:** Credential Harvesting / Social Engineering (fake subscription renewal)
- **Target:** General public (WhatsApp users, heavy concentration on Indian users per the `91` country code in the URL)
- **Sophistication:** Low-to-Medium — the premise (paid WhatsApp subscription) is easily falsifiable by anyone familiar with the app, but the HTML styling imitates WhatsApp's green branding convincingly. The attack likely succeeds against less tech-savvy or elderly users.
- **MITRE ATT&CK:**
  - T1566.002 — Phishing: Spearphishing Link
  - T1598.003 — Phishing for Information: Spearphishing Link

---

## Step 8 — Recommended Actions

- [x] Block sender domain `whatsapp-renew-service.com` at email gateway
- [x] Block sender IP `178.62.105.44` at firewall (and monitor the /24 at DigitalOcean London)
- [x] Add phishing URL to proxy blocklist
- [x] Submit to PhishTank and report to DigitalOcean abuse (abuse@digitalocean.com) with .eml evidence
- [x] Alert users that WhatsApp is free and does **not** have a paid subscription — any email claiming otherwise is fraud
- [x] If any user clicked: have them verify account integrity in the WhatsApp app (Settings → Linked Devices) and enable two-step verification

---

## Screenshots — Case 04

### AbuseIPDB (178.62.105.44)
![AbuseIPDB results for 178.62.105.44](screenshots/case-04/abuseipdb.png)

> IP hosted at DigitalOcean London (AS14061), United Kingdom. **Reported 5 times** — unlike the previous cases' freshly provisioned hosts, this IP has prior abuse history, indicating the attacker is reusing rented infrastructure.

---

### MXToolbox (whatsapp-renew-service.com)
![MXToolbox results for whatsapp-renew-service.com](screenshots/case-04/mxtoolbox.png)

> 10 critical problems: no HTTP resolution, no DNS records, no MX, no SPF, no DMARC. Consistent with the pattern across all cases — a disposable domain with no legitimate mail infrastructure.

---

### URLScan.io (whatsapp-renew-service.com — live scan)

> *URLScan.io returned HTTP 400 — DNS Error - Could not resolve domain. The domain `whatsapp-renew-service.com` could not be resolved to a valid IPv4/IPv6 address at time of scanning.* (Screenshot was replaced during upload; see URL Analysis above for verdict.)

---

### VirusTotal — URL Scan
![VirusTotal URL scan 0/91](screenshots/case-04/virustotal-url.png)

> 0/91 vendors flagged the URL `https://whatsapp-renew-service.com/renew?phone=919876543210`. The URL itself evaded detection, but see the IP scan below for more signal.

---

### VirusTotal — IP Scan (178.62.105.44)
![VirusTotal IP scan 2/94 Malicious](screenshots/case-04/virustotal-ip.png)

> **2/94 vendors (Criminal IP and SOCRadar) flagged the IP as malicious.** This is the first case in the series where external threat intelligence has positive hits — the DigitalOcean IP has been reused across multiple campaigns long enough to accumulate reputation data.

---

### WHOIS (IP 178.62.105.44)

> *IP WHOIS returns RIPE NCC block allocation (178.0.0.0/8, RegDate 2009-01-30, Amsterdam NL). The actual tenant per AbuseIPDB is DigitalOcean, LLC — RIPE data reflects the regional registry, not the current operator.* (Screenshot was replaced during upload.)

---
---

# Case 05 — Fake HDFC Credit Card Alert

## Raw Sample: `05-fake-hdfc-credit-card-alert.eml`

```eml
From: HDFC Bank Alerts <transaction-alert@hdfc-bankalerts-secure.in>
To: customer@gmail.com
Subject: ALERT: Suspicious Transaction of Rs. 49,999 on Your HDFC Credit Card
Date: Fri, 18 Apr 2026 16:20:00 +0530
Message-ID: <20260418162000.MNO345@hdfc-bankalerts-secure.in>
MIME-Version: 1.0
Content-Type: text/html; charset="UTF-8"
Return-Path: <noreply@hdfc-bankalerts-secure.in>
Received: from hdfc-bankalerts-secure.in (45.33.92.117) by mx.google.com with SMTP;
        Fri, 18 Apr 2026 16:20:01 +0530
Authentication-Results: mx.google.com;
        spf=fail (sender IP is 45.33.92.117)
        dkim=none
        dmarc=fail
```

---

# Investigation: ALERT: Suspicious Transaction of Rs. 49,999 on Your HDFC Credit Card

## Quick Summary

| Field | Value |
|---|---|
| Sample file | `05-fake-hdfc-credit-card-alert.eml` |
| Date received | 2026-04-18 |
| Subject | ALERT: Suspicious Transaction of Rs. 49,999 on Your HDFC Credit Card |
| Sender (display name) | HDFC Bank Alerts |
| Sender (actual email) | transaction-alert@hdfc-bankalerts-secure.in |
| Sender IP | 45.33.92.117 |
| Attack type | Credential Harvesting / Financial Fraud (panic-inducing fake fraud alert) |
| Brand impersonated | HDFC Bank |
| Verdict | **PHISHING** |

---

## Step 1 — Initial Triage

What I noticed in the first 30 seconds:

- **Urgency language?** YES — *"ALERT"* in subject, *"Suspicious Transaction Detected"*, *"click below immediately to block your card"*
- **Threatening consequences?** YES — implies Rs. 49,999 has already been charged; forces panic-driven action
- **Suspicious sender address?** YES — sent from `hdfc-bankalerts-secure.in`; HDFC Bank's real domains are `hdfcbank.com` and `hdfcbank.net`
- **Reverse-psychology hook?** YES — *"If this was your transaction, you can safely ignore this email"* is a classic social engineering line that makes the victim feel the email is legitimate for framing it as optional
- **Links present?** YES — 1 phishing link to `https://hdfc-bankalerts-secure.in/fraud-report?card=7823`
- **Attachments?** None

---

## Step 2 — Header Analysis

Key headers from the `.eml` file:

- **From:** HDFC Bank Alerts `<transaction-alert@hdfc-bankalerts-secure.in>`
- **Return-Path:** `<noreply@hdfc-bankalerts-secure.in>`
- **Received (first hop):** from hdfc-bankalerts-secure.in (45.33.92.117) by mx.google.com
- **Sender IP:** 45.33.92.117

Authentication results:

- **SPF:** FAIL (sender IP is 45.33.92.117)
- **DKIM:** NONE
- **DMARC:** FAIL

**Red flags found:**

- Return-Path doesn't match From address? **NO** — both use `hdfc-bankalerts-secure.in`, but the domain itself is fake
- Sender IP in a different country than claimed? **YES** — IP geolocates to Cedar Knolls, New Jersey, USA (Linode / Akamai); HDFC is an Indian bank
- X-Mailer or unusual headers? **None visible**, but SPF/DKIM/DMARC all fail

---

## Step 3 — Sender Investigation

- **Sender domain:** `hdfc-bankalerts-secure.in`
- **WHOIS registration date:** IP WHOIS shows Akamai Technologies / Linode-US block (45.33.0.0 – 45.33.127.255); domain-level WHOIS not available
- **Registrar:** Unknown for the domain; IP operator is Linode (AS63949)
- **AbuseIPDB reports:** 0 reports on IP 45.33.92.117 — freshly provisioned Linode VPS
- **VirusTotal URL check:** 0/91 vendors flagged the URL

**Analysis:** `hdfc-bankalerts-secure.in` impersonates HDFC Bank using the familiar fake-fraud-alert pattern — the scariest kind of phishing, because it exploits the victim's fear of losing money. The attacker makes the victim *want* to click urgently to "block their card", when in reality the click sends their actual card credentials to the attacker. Hosting on a US Linode VPS while impersonating an Indian bank is a clear geographic anomaly.

---

## Step 4 — URL Analysis

URLs found in the email body (defanged):

- `hxxps://hdfc-bankalerts-secure[.]in/fraud-report?card=7823&txn=TXN20260418`

For the phishing URL:

- **URLScan.io result:** HTTP 400 Error — DNS resolution failed; domain `hdfc-bankalerts-secure.in` could not be resolved to a valid IPv4/IPv6 address. URLScan search for the domain returned **zero community comments**, indicating this is a fresh, untracked phishing domain.
- **VirusTotal:** 0/91 vendors flagged the URL
- **Does domain match the real brand?** NO — HDFC Bank's real domain is `hdfcbank.com`
- **Redirect chain:** Domain non-resolvable at time of analysis
- **Note:** The URL embeds the last four digits of the "compromised" card (`7823`), which matches the card number shown in the email body — a social engineering technique to make the link feel personalised and legitimate

---

## Step 5 — Attachment Analysis

No attachments present.

---

## Step 6 — Extracted IOCs

| Type | Value | Context |
|---|---|---|
| Email | transaction-alert@hdfc-bankalerts-secure.in | Sender address |
| Email | noreply@hdfc-bankalerts-secure.in | Return-Path |
| IP | 45.33.92.117 | Sending server (Cedar Knolls, NJ, USA — Linode / AS63949) |
| Domain | hdfc-bankalerts-secure.in | Phishing domain impersonating HDFC Bank |
| URL | hxxps://hdfc-bankalerts-secure[.]in/fraud-report?card=7823&txn=TXN20260418 | Phishing link in email body |

---

## Step 7 — Attack Classification

- **Type:** Credential Harvesting / Financial Fraud — the landing page almost certainly collects full card number, CVV, expiry, and net-banking credentials under the guise of "blocking the card"
- **Target:** HDFC Bank credit card holders (Indian retail banking customers)
- **Sophistication:** High — uses panic framing (Rs. 49,999 foreign transaction in Dubai), credible branding (HDFC CIN number, real 24x7 helpline number `1800-266-4332`), and reverse-psychology reassurance (*"If this was your transaction, you can safely ignore"*). This is one of the most dangerous patterns because victims actively *want* to click the link.
- **MITRE ATT&CK:**
  - T1566.002 — Phishing: Spearphishing Link
  - T1598.003 — Phishing for Information: Spearphishing Link
  - T1056 — Input Capture (card data harvesting on landing page)

---

## Step 8 — Recommended Actions

- [x] Block sender domain `hdfc-bankalerts-secure.in` at email gateway
- [x] Block sender IP `45.33.92.117` at firewall and monitor Linode /16 range for related infrastructure
- [x] Add phishing URL to proxy blocklist
- [x] Report to Linode abuse (abuse@linode.com) with .eml evidence
- [x] Submit to PhishTank and report to RBI Sachet (sachet.rbi.org.in) and cybercrime.gov.in
- [x] Alert users: HDFC fraud reporting is done via the **PhoneBanking number on the back of the card** or through the official HDFC mobile app — **never via email link**
- [x] If any user clicked and entered card details: contact HDFC PhoneBanking (1800-266-4332) immediately to block the card and dispute any unauthorised transactions; file cybercrime complaint at 1930 or cybercrime.gov.in

---

## Screenshots — Case 05

### AbuseIPDB (45.33.92.117)
![AbuseIPDB results for 45.33.92.117](screenshots/case-05/abuseipdb.png)

> IP hosted at Linode (AS63949), Cedar Knolls, New Jersey, USA. 0 abuse reports — freshly provisioned Linode VPS, no prior reputation data. Use of US hosting to impersonate an Indian bank is a clear geographic anomaly.

---

### MXToolbox (hdfc-bankalerts-secure.in)
![MXToolbox results for hdfc-bankalerts-secure.in](screenshots/case-05/mxtoolbox.png)

> 10 critical problems: no HTTP, no DMARC, no SPF, no MX, no DNS. The fake HDFC domain has zero legitimate email infrastructure — confirming it exists solely for phishing.

---

### URLScan.io (hdfc-bankalerts-secure.in — search)
![URLScan.io search for hdfc-bankalerts-secure.in](screenshots/case-05/urlscan.png)

> URLScan community search returned zero results for `hdfc-bankalerts-secure.in` — the domain has no public scan history. A live scan of the phishing URL returned HTTP 400 (DNS Error - Could not resolve domain), confirming the site is offline or not hosted at DNS resolution time.

---

### VirusTotal — URL & Domain Scan

> *VirusTotal reported 0/91 vendor detections for both the domain `hdfc-bankalerts-secure.in` and the URL `https://hdfc-bankalerts-secure.in/fraud-report?card=7823`. As with the earlier cases, zero VT detections on a newly registered look-alike domain is **not** an indicator of safety.* (Screenshots were replaced during upload; see Step 4 for verdict.)

---

### WHOIS (IP 45.33.92.117)

> *IP WHOIS shows Akamai Technologies, Inc. LINODE-US (NET-45-33-0-0-1) covering 45.33.0.0 – 45.33.127.255, with Linode LINODE (NET-45-33-0-0-2) as the sub-assignment. The IP is operated by Linode (now owned by Akamai).* (Screenshot was replaced during upload.)

---

## Cross-Case Summary

| | Case 01 — SBI KYC | Case 02 — Amazon | Case 03 — Income Tax | Case 04 — WhatsApp | Case 05 — HDFC Card |
|---|---|---|---|---|---|
| Phishing domain | sbi-secure-verify.in | amazon-notifications-in.com | incometax-gov-refund.in | whatsapp-renew-service.com | hdfc-bankalerts-secure.in |
| Sender IP | 185.234.72.19 | 91.215.41.33 | 103.74.19.88 | 178.62.105.44 | 45.33.92.117 |
| Sender IP country | 🇩🇪 Germany | 🇷🇺 Russia | 🇮🇳 India | 🇬🇧 UK | 🇺🇸 USA |
| Hosting provider | DeinServerHost | DDOS-GUARD LTD | Zess Networks | DigitalOcean | Linode/Akamai |
| SPF | FAIL | FAIL | FAIL | FAIL | FAIL |
| DKIM | NONE | NONE | NONE | NONE | NONE |
| DMARC | FAIL | FAIL | FAIL | FAIL | FAIL |
| VirusTotal URL | 0/91 | 0/91 | 0/91 | 0/91 | 0/91 |
| VirusTotal IP | — | — | — | **2/94 malicious** | — |
| AbuseIPDB reports | 0 | 0 | 0 | **5** | 0 |
| Site resolvable? | No | No | No | No | No |
| Sophistication | Medium | Medium | **High** | Low-Medium | **High** |
| Attack type | Credential Harvesting | Credential Harvesting | Credential Harvesting + Financial Fraud | Credential Harvesting | Credential Harvesting + Financial Fraud |

> **Key insights across all five cases:**
>
> 1. **Email authentication uniformly fails.** Every single sample fails SPF, has no DKIM signature, and fails DMARC. A correctly configured mail gateway would quarantine or reject all five at the MX layer.
> 2. **Threat feeds lag reality.** All five URLs return 0/91 on VirusTotal despite being unambiguously malicious. Only Case 04's reused DigitalOcean IP has independent vendor detections (2/94). Blocklist-based defence alone is insufficient — behavioural and header-based detection is essential.
> 3. **Hosting spans five countries across five cases**, showing attackers rent short-lived VPS capacity worldwide rather than operating from one location. Geographic anomaly (e.g., "Indian bank email sent from Russia") remains one of the strongest header-level signals.
> 4. **The Indian-hosted attack (Case 03) is the most dangerous** because it defeats the geographic-anomaly heuristic. Defences must go beyond "sender IP country matches claimed brand".
> 5. **Look-alike domain patterns are consistent:** insertion of `-secure`, `-verify`, `-service`, `-renew`, `-alerts`, `-gov-refund`, or `-notifications` between the brand name and the TLD. A gateway regex watching for these patterns alongside known brand names would catch most of this campaign class.
