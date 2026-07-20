# 🔎 Search Skills — Reconnaissance & OSINT Notes

![Category](https://img.shields.io/badge/Category-Reconnaissance-orange) ![Type](https://img.shields.io/badge/Type-OSINT%20%2F%20Search%20Skills-blueviolet)

Notes and practical commands on advanced search techniques, technology fingerprinting, and DNS reconnaissance — core skills used during the information-gathering phase of a penetration test.

---

## 1️⃣ Google Dorking (Advanced Search Operators)

Google Dorks use special search operators to filter results and surface information that isn't easily found through a normal search — commonly used in the passive reconnaissance phase to discover exposed files, admin pages, and sensitive data indexed by search engines.

| # | Dork | Purpose |
|---|---|---|
| 1 | `site:google.com` | Restrict results to a specific domain |
| 2 | `site:google.com "xbox"` | Search for a specific keyword within a domain |
| 3 | `site:google.com filetype:pdf/sql/txt/xlsx` | Find specific file types hosted on a domain (potential leaked documents/backups) |
| 4 | `site:google.com intitle:"Call of duty"` | Find pages where the keyword appears in the page **title** |
| 5 | `site:google.com inurl:"Call of duty"` | Find pages where the keyword appears in the **URL** |
| 6 | `site:google.com intext:"Call of duty"` | Find pages where the keyword appears in the **body text** |
| 7 | `index of movies` | Finds open directory listings — occurs when a website is missing an `index.html`/`index.php` file and the Apache web server auto-generates a file listing instead |
| 8 | `site:linkedin.com/in "Microsoft.com"` | OSINT technique to find employees of a company on LinkedIn — commonly used to identify HR/staff for social engineering awareness or contact mapping |

**Key takeaway:** Dorking is a passive recon technique — no direct interaction with the target's servers, just smarter querying of what's already publicly indexed.

---

## 2️⃣ Username OSINT — Sherlock

```bash
sherlock "username"
```

Searches a given username across a large number of social media platforms and websites simultaneously, helping map out a target's online footprint from a single identifier.

---

## 3️⃣ Identifying Website Tech (Backend Fingerprinting)

Knowing what technologies power a website (CMS, frameworks, server software, versions) helps identify known/public exploits relevant to that specific stack.

| Tool | Notes |
|---|---|
| **Wappalyzer** (Browser Extension) | Install from browser settings/extensions. Instantly reveals the backend technologies and their versions used by a site. Version info can then be used to search for known exploits. |
| **BuiltWith** | Web-based alternative to Wappalyzer — cross-checking multiple tools gives a more complete tech-stack picture during footprinting. |
| **WhatWeb** | Kali Linux command-line tool for identifying web technologies; also commonly reveals the server's IP address. |

```bash
whatweb example.com
```

---

## 4️⃣ WHOIS Lookups

WHOIS reveals registration details about a domain — useful for identifying who owns/manages a target and when key domain events occurred.

```bash
whois google.com
```

**Information typically retrieved:**
- Domain registrar
- Registry creation date
- Last updated date
- Expiry date
- Registrant's postal code, city, email
- Admin name and contact details

*Works both via command line (Linux) and through WHOIS lookup websites.*

---

## 5️⃣ DNS Reconnaissance

DNS records are stored as zone files on DNS servers and reveal a lot about how a domain's infrastructure is configured.

### Core Commands

```bash
# Get IPv4/IPv6 address of a domain
host google.com

# Get domain registrar info
whois google.com

# Find the domain's Name Servers (NS)
host -t ns google.com

# Find the domain's Mail Servers (MX)
host -t mx google.com

# Get IPv4/IPv6 addresses (works on Windows too)
nslookup google.com

# Interactive nslookup — query specific record types
nslookup
> set type=CNAME
> google.com
> set type=AAAA
> google.com
```

### Using `dig` (DNS lookup utility)

```bash
dig google.com
dig google.com -t SOA
dig google.com -t TXT
```

> **Note:** `nslookup` and `dig` are both dedicated **DNS query tools**, but `dig` typically provides more detailed, raw output — preferred for deeper DNS troubleshooting and recon.

### DNS Record Types Reference

| Query Type | Result |
|---|---|
| **A** | IPv4 address(es) for the domain |
| **AAAA** | IPv6 address(es) for the domain |
| **CNAME** | Canonical Name — an alias that points one domain name to another |
| **MX** | Mail Servers — the servers responsible for handling email for the domain |
| **SOA** | Start of Authority — the primary name server, admin email, and zone serial number |
| **TXT** | Text Records — arbitrary text, commonly used for SPF, DKIM, DMARC, and domain verification |

---

## 🧠 Skills Practiced

`Google Dorking` `OSINT` `Username Enumeration` `Technology Fingerprinting` `WHOIS Lookups` `DNS Enumeration` `Passive Reconnaissance`

---

## 📌 Source

These notes were compiled from independent study via YouTube tutorials alongside the TryHackMe learning path — not taken directly from a TryHackMe room, but supplementing the **Search Skills** section of the Cyber Security Journey.
