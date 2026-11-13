# PENETRATION TESTING REPORT

### Footprinting & Network Scanning Phases

**W2-PM-FINAL | CYBERSECURITY | NETWORKWALKS**

| Field | Detail |
|---|---|
| **Pentester Name** | Chinedu Okoli |
| **Program / Batch** | B084, Networkwalks Cybersecurity Internship |
| **Date** | 13 November 2026 |
| **Modules completed** | W2-PM1 (Footprinting with Multiple Kali Tools)<br>W2-PM5 (Zenmap Network Scanning) |
| **Client / Target** | 1. Networkwalks (`networkwalks.com`, written permission on file)<br>2. My own local LAN |
| **Permission secured from client?** | Yes  |
| **Phases covered** | Phase 1: Reconnaissance & Footprinting<br>Phase 2: Scanning & Network Discovery |

---

## 1. Liability Disclaimer

All activities in this report were carried out only on systems where I hold written permission (`networkwalks.com` and its DNS records) or on devices and networks I own (my own LAN). This material is for education and research only. Unauthorised access is a crime in most countries, even when nothing is damaged.

## 2. Introduction

This report documents Week 2 of the Networkwalks Cybersecurity Internship. W2-PM1 covers the footprinting phase: gathering public information about `networkwalks.com` with six Kali Linux tools. W2-PM5 covers the scanning phase: discovering live hosts on my own local network with Zenmap. Together they show how an attacker moves from public information to a map of live systems, and how a defender can use the same steps to see their own exposure.

Every step includes the command used, what I observed, a screenshot as evidence, and why the finding matters.

## 3. Tools Used

| Tool | Purpose |
|---|---|
| Kali Linux | Environment for footprinting tools |
| whois | Domain registration details (registrar, dates, name servers) |
| whatweb | Fingerprint web technologies (server, CMS, plugins, IP) |
| nslookup | Resolve the domain to its IP address |
| curl -I | Read HTTP response headers |
| wafw00f | Detect a Web Application Firewall |
| dnsrecon | Enumerate DNS records (NS, MX, SPF/TXT, SRV) |
| Zenmap (Nmap GUI) | Discover live hosts, IPs and MAC addresses on my LAN |
| Windows CMD (`ipconfig`) | Identify my local IP and subnet |

## 4. Activities Performed

### 4.1 Footprinting & Reconnaissance (W2-PM1)

Target: `networkwalks.com`. All six commands were run from Kali Linux.

| Tool | Command | Key finding (from my own output) |
|---|---|---|
| whois | `whois networkwalks.com` | Registrar GoDaddy; created 2019-11-06, expires 2027-11-06; name servers NS6135/NS6136.HOSTGATOR.COM; DNSSEC unsigned; registrant details hidden by privacy service |
| whatweb | `whatweb networkwalks.com` | Apache; WordPress 7.1.4; WP Download Manager 3.3.76; jQuery 3.7.1; Google Tag Manager; IP 192.232.216.135; email info@networkwalks.com exposed; HTTP redirects to HTTPS |
| nslookup | `nslookup networkwalks.com` | 192.232.216.135 (via resolver 8.8.8.8, non-authoritative answer) |
| curl | `curl -I https://networkwalks.com` | HTTP/1.1 200 OK; `Server: Apache`; WordPress REST API `/wp-json/` advertised in Link header; `__wpdm_client` cookie (Secure, HttpOnly); `X-nginx-cache: WordPress` and `X-Endurance-Cache-Level` headers |
| wafw00f | `wafw00f networkwalks.com` | Yes: ModSecurity (SpiderLabs), detected with 2 requests (wafw00f v2.3.2) |
| dnsrecon | `dnsrecon -d networkwalks.com` | MX mail.networkwalks.com; A 192.232.216.135; SPF `~all` (soft fail); Google site-verification TXT; BIND 9.16.23-RH on both name servers; 8 cPanel autodiscover SRV records; DNSSEC query returned no answer |

In W2-PM1, I footprinted networkwalks.com with six Kali Linux tools, using only public information. WHOIS showed that the domain is registered with GoDaddy, that its name servers belong to HostGator, and that DNSSEC is unsigned. WhatWeb identified WordPress 7.1.4 and WP Download Manager 3.3.76. Nslookup resolved the domain to 192.232.216.135. Curl exposed the WordPress REST API endpoint /wp-json/. Wafw00f detected ModSecurity (SpiderLabs), and DNSRecon revealed the mail server, an SPF policy ending in ~all, cPanel service records and BIND 9.16.23-RH. None of these tools attacked the target, which shows why footprinting is quiet and why organisations should regularly review what they publish.

### 4.2 Network Scanning with Zenmap (W2-PM5)

Scope: my own local network only.

1. Found my local IP and subnet: `ipconfig` → 
2. Entered the subnet as the target in Zenmap and ran a **Ping Scan**.
3. Live hosts found: [number] → 
4. MAC addresses identified: 
5. Opened the **Topology** tab, enabled the legend and saved the topology as a PDF.


## 5. Risk Analysis / Impact

| # | Finding | Evidence | Potential impact | Risk level |
|---|---|---|---|---|
| 1 | Web technology and version exposed | whatweb: WordPress 7.1.4, WP Download Manager 3.3.76 | Versions can be matched against known advisories | Medium |
| 2 | Server IP identifiable | nslookup: 192.232.216.135 | Reveals network location of the service | Low |
| 3 | HTTP technical information exposed | curl: Apache banner, `/wp-json/` link, cache headers | Helps stack fingerprinting and enumeration | Low |
| 4 | WAF product identifiable | wafw00f: ModSecurity (SpiderLabs) | Reveals part of the security architecture | Low |
| 5 | DNS infrastructure exposed | dnsrecon: NS, MX, SPF, SRV records, BIND 9.16.23-RH | Helps build an infrastructure profile | Medium |
| 6 | Live hosts visible on LAN | Zenmap: [number] hosts | Unknown devices may be present | Medium |

These are observations, not confirmed vulnerabilities. Only information gathering and host discovery were performed. No exploitation or vulnerability validation took place.

## 6. Recommendations

1. Review what technology and version information is publicly visible, and remove version strings where possible.
2. Keep the CMS, plugins and other web software updated.
3. Review HTTP response headers and restrict unneeded endpoints.
4. Review DNS records regularly and publish only what is required.
5. Keep the WAF enabled and tuned.
6. Periodically scan your own network to keep an inventory of active devices.
7. Investigate any unknown device found on the network.
8. Keep network documentation and topology diagrams up to date.
9. Perform security testing only with authorisation.

## 7. Conclusion

The main lesson is that information gathering is the foundation of everything that follows. Software versions, DNS records, headers and live hosts are not vulnerabilities on their own, but together they give an attacker a map of where to look. The findings in this report are observations only, since no exploitation was attempted. From a SOC analyst's point of view, this activity is also something defenders should be able to detect, such as repeated WHOIS and DNS lookups, header requests and ping sweeps showing up in logs. Running these tools on your own assets first is a practical way to reduce exposure before someone else does.

All work was done within the authorised scope: networkwalks.com and my own LAN. I will carry these findings into the next phases of the programme.

## 8. Evidences Collected

Screenshots are stored in the `screenshots/` folder.

![whois](screenshots/_whois.png)
![whatweb](screenshots/_whatweb.png)
![nslookup](screenshots/_nslookup.png)
![curl](screenshots/_curl.png)
![wafw00f](screenshots/_wafw00f.png)
![dnsrecon](screenshots/_dnsrecon.png)
![Zenmap scan](screenshots/7_zenmap_scan.png)
![Zenmap topology](screenshots/8_zenmap_topology.png)


---
*#GraceMadeSecurity 
