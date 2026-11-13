# W2-PM1: Footprinting & Reconnaissance with Multiple Kali Tools

**Programme:** Networkwalks Cybersecurity Internship | **Week 2, Project Module 1**
**Target:** `networkwalks.com` (the programme's own site, set as the lab target in the brief)
**Environment:** Kali Linux VM (VirtualBox), NAT network
**Scope:** Passive/low-impact reconnaissance only. No exploitation, no scanning beyond the six tools below. For educational use.

---

## Objective

Build a profile of a live target using only publicly available information, and explain how an attacker would use each finding and how a defender would reduce it.

## Folder structure

```
W2-PM1-Footprinting/
├── README.md
├── screenshots/
│   ├── 01_whois.png
│   ├── 02_whatweb.png
│   ├── 03_nslookup.png
│   ├── 04_curl_headers.png
│   ├── 05_wafw00f.png
│   └── 06_dnsrecon.png
└── outputs/
    ├── 01_whois.txt
    ├── 02_whatweb.txt
    ├── 03_nslookup.txt
    ├── 04_curl_headers.txt
    ├── 05_wafw00f.txt
    └── 06_dnsrecon.txt
```

## Setup: run all six tools and save each output

```bash
mkdir -p ~/W2-PM1-Footprinting/{screenshots,outputs} && cd ~/W2-PM1-Footprinting

whois networkwalks.com              | tee outputs/01_whois.txt
whatweb --color=never networkwalks.com | tee outputs/02_whatweb.txt
nslookup networkwalks.com           | tee outputs/03_nslookup.txt
curl -I https://networkwalks.com    | tee outputs/04_curl_headers.txt
wafw00f networkwalks.com            | tee outputs/05_wafw00f.txt
dnsrecon -d networkwalks.com        | tee outputs/06_dnsrecon.txt
```

Take a screenshot of each terminal run (right after the command finishes) and save it into `screenshots/` using the names above. `--color=never` keeps the whatweb text file free of colour codes.

---

## Task 1: whois (domain registration)

**Command:** `whois networkwalks.com`
**Evidence:** `screenshots/01_whois.png`, `outputs/01_whois.txt`

**What to record from your output:** registrar, creation date, registry expiry date, name servers, domain status flags, DNSSEC status, abuse contact.

**Attacker's view:** The name servers show the hosting provider, and dates and contacts help with planning and social engineering.
**Defender's view:** Use WHOIS privacy where available, keep registrar-lock statuses on, and consider enabling DNSSEC.

## Task 2: whatweb (technology fingerprinting)

**Command:** `whatweb networkwalks.com`
**Evidence:** `screenshots/02_whatweb.png`, `outputs/02_whatweb.txt`

**What to record:** HTTP status/redirect (HTTP to HTTPS), web server, CMS and version, plugins, JavaScript libraries, server IP, any exposed email address.

**Attacker's view:** Exact software versions can be matched against vulnerability databases. A leaked IP and email support further targeting.
**Defender's view:** Keep the CMS and plugins patched, remove version strings and generator meta tags, and avoid publishing raw email addresses (use a contact form).

## Task 3: nslookup (DNS resolution)

**Command:** `nslookup networkwalks.com`
**Evidence:** `screenshots/03_nslookup.png`, `outputs/03_nslookup.txt`

**What to record:** the resolver used, the "non-authoritative answer" and the resolved IP address.

**Attacker's view:** The IP lets them look at the hosting network, other sites on the same IP, and the wider infrastructure.
**Defender's view:** Put a CDN or reverse proxy in front of the origin so the real IP is not exposed directly.

## Task 4: curl -I (HTTP response headers)

**Command:** `curl -I https://networkwalks.com`
**Evidence:** `screenshots/04_curl_headers.png`, `outputs/04_curl_headers.txt`

**What to record:** status line, `server` banner, caching headers, cookies and their flags (Secure, HttpOnly), `link` headers revealing endpoints such as the WordPress REST API (`/wp-json/`), referrer and permissions policies.

**Attacker's view:** Headers fingerprint the stack and point to entry points without loading the page.
**Defender's view:** Trim the server banner, restrict or protect unneeded API endpoints, and review security headers (for example HSTS and Content-Security-Policy).

## Task 5: wafw00f (WAF detection)

**Command:** `wafw00f networkwalks.com`
**Evidence:** `screenshots/05_wafw00f.png`, `outputs/05_wafw00f.txt`

**What to record:** whether a WAF was detected, which product, and the number of requests sent.

**Attacker's view:** A WAF means naive attempts will be blocked or logged, so an attacker must adapt.
**Defender's view:** A WAF is a useful layer, but it is not a substitute for patching. Keep rules tuned and monitor its logs.

## Task 6: dnsrecon (DNS enumeration)

**Command:** `dnsrecon -d networkwalks.com`
**Evidence:** `screenshots/06_dnsrecon.png`, `outputs/06_dnsrecon.txt`

**What to record:** SOA and NS records, mail (MX) server, A records, TXT records (SPF, site-verification), SRV records (for example cPanel autodiscovery), DNS software version if disclosed, DNSSEC result, total records found.

**Attacker's view:** The DNS map reveals mail and hosting setup, service endpoints and software versions that can be searched for known weaknesses.
**Defender's view:** Hide DNS software versions, review the SPF policy (a hard fail `-all` is stricter than a soft fail `~all`), remove stale or unneeded records, and enable DNSSEC if supported.

---

## Combined target profile

Fill this table from **your own** outputs.

| Category | Finding | Source tool |
|---|---|---|
| Registrar | GoDaddy.com, LLC (registrant hidden by privacy service) | whois |
| Created / expires | 2019-11-06 / 2027-11-06 | whois |
| Name servers / host | NS6135 / NS6136.HOSTGATOR.COM | whois, dnsrecon |
| Server IP | 192.232.216.135 | nslookup, whatweb |
| Web server | Apache | whatweb, curl |
| CMS and version | WordPress 7.1.4 | whatweb |
| Plugins / libraries | WP Download Manager 3.3.76, jQuery 3.7.1, Google Tag Manager | whatweb |
| Exposed endpoints | /wp-json/ (WordPress REST API) | curl |
| Cookies and flags | __wpdm_client (Secure, HttpOnly) | curl |
| WAF | ModSecurity (SpiderLabs) | wafw00f |
| Mail server | mail.networkwalks.com (192.232.216.135) | dnsrecon |
| SPF policy | v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all (soft fail) | dnsrecon |
| DNS software version | BIND 9.16.23-RH | dnsrecon |
| DNSSEC | Unsigned | whois, dnsrecon |

## Key takeaways

1. Everything above came from public information, and the target was never attacked, which is why footprinting is hard to detect.
2. The more an organisation leaks (versions, IPs, emails, endpoints), the easier every later stage becomes.
3. Defenders run these same tools on their own assets to see what an attacker sees, then reduce it.
4. These findings feed the scanning and assessment work in the next modules.

## Ethics & scope

All work was done against the target designated in the Networkwalks task brief, using read-only reconnaissance tools. Hacking is only legal on systems you own, in a lab, or with written permission under an agreed scope.

---
*Author: Chinedu Okoli | #GraceMadeSecurity #TSAcademy*
