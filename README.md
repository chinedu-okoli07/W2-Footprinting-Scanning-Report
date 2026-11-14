# W2-PM1: Footprinting & Reconnaissance with Multiple Kali Tools

**Programme:** Networkwalks Cybersecurity Internship | **Week 2, Project Module 1**
**Target:** `networkwalks.com` (the programme's own site, set as the lab target in the task brief)
**Environment:** Kali Linux VM (VirtualBox) | **Date run:** 13 November 2026
**Scope:** Passive and low-impact reconnaissance only. No exploitation. For educational use.

This file gives the tool-by-tool detail behind Section 4.1 of the main report ([README](README.md)).

---

## Objective

Build a profile of a live target using only publicly available information, and explain how an attacker could use each finding and how a defender can reduce it.

## Commands used

```bash
whois networkwalks.com
whatweb networkwalks.com
nslookup networkwalks.com
curl -I https://networkwalks.com
wafw00f networkwalks.com
dnsrecon -d networkwalks.com
```

---

## Task 1: whois (domain registration)

**Command:** `whois networkwalks.com`

**Findings:**
- Registrar: GoDaddy.com, LLC
- Created 2019-11-06, registry expiry 2027-11-06, last updated 2025-11-12
- Name servers: NS6135.HOSTGATOR.COM and NS6136.HOSTGATOR.COM
- DNSSEC: unsigned
- Domain status: delete, renew, transfer and update locks are all on
- Registrant details are hidden behind a privacy service (Domains By Proxy)

![whois](screenshots/1_whois.png)
![whois registrant](screenshots/1b_whois_registrant.png)

**Attacker's view:** The name servers show the hosting provider, and dates and contacts help with planning and social engineering.
**Defender's view:** Registrar locks and privacy are already in place, which is good. Consider enabling DNSSEC.

## Task 2: whatweb (technology fingerprinting)

**Command:** `whatweb networkwalks.com`

**Findings:**
- HTTP redirects (301) to HTTPS
- Web server: Apache
- CMS: WordPress 7.1.4, with WP Download Manager 3.3.76
- jQuery 3.7.1, Google Tag Manager, Open Graph metadata
- Server IP: 192.232.216.135
- Public email address exposed: `info@networkwalks.com`
- Page title: Networkwalks Academy

![whatweb](screenshots/2_whatweb.png)

**Attacker's view:** Exact software versions can be matched against vulnerability databases. A leaked IP and email support further targeting.
**Defender's view:** Keep the CMS and plugins patched, remove version strings and generator meta tags, and use a contact form instead of publishing a raw email address.

## Task 3: nslookup (DNS resolution)

**Command:** `nslookup networkwalks.com`

**Findings:** Resolver 8.8.8.8, non-authoritative answer, resolved IP **192.232.216.135**.

![nslookup](screenshots/3_nslookup.png)

**Attacker's view:** The IP lets an attacker look at the hosting network and the wider infrastructure.
**Defender's view:** A CDN or reverse proxy in front of the origin would avoid exposing the real IP directly.

## Task 4: curl -I (HTTP response headers)

**Command:** `curl -I https://networkwalks.com`

**Findings:**
- HTTP/1.1 200 OK, `Server: Apache`
- `Link` header advertises the WordPress REST API at `/wp-json/`
- Cookie `__wpdm_client` set with Secure and HttpOnly flags
- Caching headers `X-Nginx-Cache: WordPress` and `X-Endurance-Cache-Level: 0`
- `Referrer-Policy` and `Permissions-Policy` are present
- HSTS, Content-Security-Policy and X-Frame-Options did not appear in the response headers

![curl](screenshots/4_curl.png)

**Attacker's view:** Headers fingerprint the stack and point to entry points without loading the page.
**Defender's view:** Trim the server banner, restrict unneeded API endpoints, and add security headers such as HSTS and Content-Security-Policy.

## Task 5: wafw00f (WAF detection)

**Command:** `wafw00f networkwalks.com`

**Findings:** The site is behind **ModSecurity (SpiderLabs)**, detected with 2 requests.

![wafw00f](screenshots/5_wafw00f.png)

**Attacker's view:** A WAF means naive attempts will be blocked or logged, so an attacker must adapt.
**Defender's view:** A WAF is a useful layer, but it does not replace patching. Keep its rules tuned and monitor its logs.

## Task 6: dnsrecon (DNS enumeration)

**Command:** `dnsrecon -d networkwalks.com`

**Findings:**
- SOA and NS: ns6135.hostgator.com (50.87.144.87) and ns6136.hostgator.com (192.232.216.131)
- DNS software: BIND 9.16.23-RH on both name servers
- MX: mail.networkwalks.com (192.232.216.135); A record 192.232.216.135
- SPF: `v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all` (soft fail)
- Google site-verification TXT record
- 8 cPanel autodiscover SRV records (the "8 Records Found" line counts these SRV records)
- DNSSEC query returned no answer

![dnsrecon](screenshots/6_dnsrecon.png)

**Attacker's view:** The DNS map reveals mail and hosting setup, service endpoints and software versions that can be searched for known weaknesses.
**Defender's view:** Hide DNS software versions, consider a hard-fail SPF (`-all`), remove unneeded records, and enable DNSSEC if supported.

---

## Combined target profile

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
*Author: Chinedu Okoli | #GraceMadeSecurity
