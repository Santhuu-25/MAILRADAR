# MAILRADAR // Real-Time Email Intelligence & Security Analyzer

[![Python](https://img.shields.io/badge/Python-3.10%2B-25E84A.svg)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-3.0%2B-25E84A.svg)](https://palletsprojects.com/p/flask/)
[![Theme](https://img.shields.io/badge/Aesthetic-Neon%20Green%20Cybersecurity-25E84A.svg)](#design-system)
[![Google DNS](https://img.shields.io/badge/DNS-Google%20Public%20DNS%20API-25E84A.svg)](https://dns.google)

**MailRadar** is a single-page cybersecurity email intelligence platform. It performs real-time DNS authentication diagnostics, RFC 5322 syntax validation, deliverability checks, and domain security posture evaluations without requiring user login or storing private credentials.

---

## 🖥️ Visual Design System

- **Background:** Deep black (`#080B09`) with an ambient cyber grid.
- **Accents:** High-voltage neon green (`#25E84A`) with subtle glowing borders.
- **Analysis Cards:** Dark grey surfaces (`#111612` / `#161D17`) with hairline boundaries (`#1C261F`).
- **Typography:** Technical monospace (`JetBrains Mono`) for data, SHA hashes, and DNS records + clean sans (`Inter`) for structure.
- **Loading State:** Live circular radar scanning sweep with pulsing target rings.
- **Single Page Architecture:** Zero navigation pages, zero user logins, zero sidebars — maximum information density.

---

## 🔍 Core Features & Diagnostics

1. **Email Syntax Inspection:**
   - Strict RFC 5322 structural compliance check.
   - Local mailbox length (&le; 64 chars) and total length (&le; 254 chars) verification.
   - Illegal character detection, double-dot (`..`) mitigation, and malformed label checks.

2. **Domain Resolution (A Records):**
   - Real-time DNS lookups via **Google Public DNS JSON API** (`https://dns.google/resolve`).
   - Identifies non-existent domains (NXDOMAIN / Status 3) immediately.

3. **Mail Exchanger (MX Records):**
   - Discovers all configured inbound mail servers and their integer priorities.
   - Highlights primary mail exchanger host.

4. **Sender Policy Framework (SPF):**
   - Inspects domain TXT records specifically for `v=spf1`.
   - **Distinct Record Handling:** Does not confuse a missing SPF record with a missing TXT record (if TXT exists for verification but lacks `v=spf1`, SPF is labeled **Not Found** while TXT records are acknowledged).
   - Evaluates qualifier policies: Hardfail (`-all`), Softfail (`~all`), Neutral (`?all`), Permissive (`+all`).

5. **DMARC Policy Enforcement:**
   - Queries `_dmarc.<domain>` via Google Public DNS.
   - Extracts policy directives: `p=reject`, `p=quarantine`, or `p=none`.
   - Displays percentage coverage (`pct`) and aggregate reporting endpoints (`rua`).

6. **Abstract Email Validation API Integration (Real API):**
   - Connects to `https://emailvalidation.abstractapi.com/v1/` when `ABSTRACT_API_KEY` is configured.
   - Evaluates:
     - Deliverability verdict (`DELIVERABLE`, `UNDELIVERABLE`, `UNKNOWN`).
     - Quality score (0.00 – 1.00).
     - SMTP handshake validity.
     - Disposable / temporary mailbox status.
     - Free webmail vs. private corporate domain classification.
   - If unconfigured, clearly displays: `"Abstract API: Not configured in .env."`

7. **EmailRep Reputation Intelligence (Optional):**
   - Connects to `https://emailrep.io/<email>` using `EMAILREP_API_KEY`.
   - Activated strictly when an authorized key is provided.
   - If no key is set, explicitly displays: `"Email reputation: Not configured."` (Never relies on unauthenticated access).

8. **Factual Standardized Labels:**
   - Strict adherence to factual labels: `Valid`, `Invalid`, `Found`, `Not Found`, `Unknown`, `API Unavailable`.
   - No fabricated scores or simulated values.

9. **Raw JSON Telemetry Export:**
   - Expandable in-browser JSON viewer with a one-click copy button for SOC incident logging.

---

## 📂 Project Structure

```
MailRadar/
├── app.py                     # Flask application, routing, and rate limiter
├── requirements.txt           # Python dependencies
├── .env.example               # Environment variables template
├── .gitignore                 # Git ignore rules (.env protected)
├── README.md                  # Documentation and setup instructions
├── services/
│   ├── __init__.py
│   ├── dns_analyzer.py        # Google Public DNS JSON API integration
│   ├── email_validator.py     # RFC 5322 syntax & Abstract API integration
│   └── reputation.py          # EmailRep API adapter
├── templates/
│   └── index.html             # Single-page cyberpunk dashboard
├── static/
│   ├── style.css              # Custom neon green dark cybersecurity styles
│   └── script.js              # Asynchronous controller & radar UI renderer
└── tests/
    └── test_analyzer.py       # 15 automated unit & integration tests
```

---

## 🔒 Security Principles

- **Zero Credential Storage:** Submitted emails are analyzed in-memory and discarded. No database, no tracking cookies.
- **Rate Limiting:** Built-in sliding window rate limiter (30 requests/minute per IP) prevents API abuse.
- **Request Timeouts:** All outbound HTTP calls enforce strict timeouts (5.0s for DNS, 8.0s for Abstract, 6.0s for EmailRep).
- **Environment Isolation:** All API keys are loaded via `python-dotenv` and excluded from source control.
- **No In-browser Key Exposure:** Frontend scripts only communicate with local `/api/analyze`; third-party keys are never sent to the client.

---

## 🛠️ Windows Installation & Run Instructions

### Step 1: Open Terminal
Open PowerShell or Command Prompt in the project folder:
```powershell
cd "c:\Users\LENOVO\OneDrive\Desktop\New folder"
```

### Step 2: Configure Environment (Optional)
Copy the example environment configuration:
```powershell
Copy-Item .env.example .env
```
*(Optional)* Edit `.env` to add your Abstract, Behind the Email, IntelBase, or Have I Been Pwned API keys:
```ini
PORT=5000
ABSTRACT_API_KEY=your_abstract_api_key_here
BEHIND_THE_EMAIL_API_KEY=your_bte_key_here
INTELBASE_API_KEY=your_intelbase_api_key_here
HIBP_API_KEY=your_hibp_api_key_here
```
> **Note:** MailRadar works out-of-the-box with Google Public DNS, dnspython, Gravatar, GitHub, and Holehe even without paid third-party API keys. Official HIBP test accounts (`@hibp-integration-tests.com`) work automatically in test mode.

### Have I Been Pwned (HIBP) Breach Checking
MailRadar integrates the official [Have I Been Pwned API v3](https://haveibeenpwned.com/API/v3):
- **Endpoint:** `POST /api/breach-check`
- **Payload:**
  ```json
  {
    "email": "account-exists@hibp-integration-tests.com",
    "authorized": true
  }
  ```
- **Requirements:** User authorization is strictly required (`authorized: true`). API keys are never exposed in responses or client code.
- **Attribution:** Data provided by [Have I Been Pwned](https://haveibeenpwned.com) under Creative Commons Attribution 4.0 International License.

### IntelBase API Whitelisting
IntelBase requires lookup requests to originate from an authorized IP address.
Check your machine egress IP using:
```bash
curl http://127.0.0.1:5000/api/intelbase/ip
```
Add the returned IP to your whitelist in the [IntelBase Dashboard](https://intelbase.is/dashboard).

### Step 3: Run the Application
Start the Flask server:
```powershell
py app.py
```
*(or `python app.py`)*

### Step 4: Open in Your Browser
Navigate to:
👉 **http://127.0.0.1:5000**

---

## 🧪 Running Automated Tests

MailRadar includes a test suite covering syntax, DNS resolution, SPF/DMARC handling, timeouts, invalid keys, and partial API failures:

```powershell
py -m unittest tests/test_analyzer.py
```

### Test Coverage Checklist:
- [x] Valid email syntax
- [x] Invalid email syntax (malformed domains, missing `@`, consecutive dots)
- [x] Missing MX records
- [x] Missing SPF records (distinct from missing TXT records)
- [x] Missing DMARC records
- [x] DNS API timeout handling
- [x] Abstract API timeout handling
- [x] Invalid API key handling (HTTP 401)
- [x] API quota exceeded handling (HTTP 429)
- [x] Partial API failure (DNS succeeds while Abstract fails)
- [x] Health check and API validation endpoints

