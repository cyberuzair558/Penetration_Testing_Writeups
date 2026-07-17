# 🌐 DNS in Detail — TryHackMe Writeup

![TryHackMe](https://img.shields.io/badge/TryHackMe-DNS%20in%20Detail-red?style=for-the-badge&logo=tryhackme&logoColor=white)
![Category](https://img.shields.io/badge/Category-Networking%20%26%20Recon-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

> 📌 **Room:** [DNS in Detail](https://tryhackme.com/room/dnsindetail) — TryHackMe
> 📝 **Purpose:** Notes + quick-revision cheat sheet from solving this room, covering DNS fundamentals, record types, and the pentesting/recon relevance of DNS.

---

## 📖 Table of Contents

- [Overview](#-overview)
- [DNS Query Types](#-dns-query-types)
- [DNS Request Flow](#-dns-request-flow-resolution-process)
- [Cache Memory & TTL](#-cache-memory--ttl)
- [CNAME Records](#-cname-record)
- [Pentesting Relevance](#-pentesting-relevance)
- [DNS Enumeration Commands (Cheat Sheet)](#-dns-enumeration-commands-cheat-sheet)
- [Key Takeaway](#-key-takeaway)

---

## 🧭 Overview

DNS (Domain Name System) translates human-readable domain names into IP addresses. But beyond that basic function, DNS is a **major recon and attack surface** — from subdomain takeovers to zone transfer leaks to DNS tunneling for C2. This writeup breaks down the core concepts and the commands used to query DNS records during recon.

---

## 📋 DNS Query Types

| Query Type | Result |
|:----------:|--------|
| **A** | IPv4 address(es) for the domain |
| **AAAA** | IPv6 address(es) for the domain |
| **CNAME** | Canonical Name — an alias that points one domain name to another |
| **MX** | Mail Servers — the servers responsible for handling email for the domain |
| **SOA** | Start of Authority — the primary name server, admin email, and zone serial number |
| **TXT** | Text Records — arbitrary text, commonly used for SPF, DKIM, DMARC, and domain verification |

---

## 🔄 DNS Request Flow (Resolution Process)

1. **Local Cache** → the requesting computer checks itself first
2. **Recursive DNS Server** → ISP/custom resolver (e.g. `8.8.8.8`), which also caches results
3. **Root DNS Server** → points toward the correct TLD server
4. **TLD Server** → returns the Authoritative Nameserver for the domain
5. **Authoritative DNS Server** → returns the actual record; the answer travels back through the Recursive server to the client

```
Client → Local Cache → Recursive Resolver → Root Server → TLD Server → Authoritative Server → back to Client
```

---

## 🗄️ Cache Memory & TTL

- Resolved answers are **temporarily stored** (both locally and on the recursive server)
- **TTL (Time To Live)** = how long a cached record stays valid (in seconds)
- **Low TTL** → enables fast changes; commonly abused in **fast-flux malicious infrastructure**
- **High TTL** → stable records, fewer repeated lookups/requests

---

## 🔗 CNAME Record

- Points a domain **to another domain name** instead of directly to an IP (works like an alias)
- **Not allowed on the root/apex domain** — only valid on subdomains
- **Example:**
  ```
  blog.example.com  CNAME  ghs.googlehosted.com
  ```

---

## 🎯 Pentesting Relevance

| Attack / Technique | Description |
|---|---|
| **Subdomain Takeover** | When a service is deleted, its CNAME can be left dangling — an attacker registers the service and claims the subdomain (Tools: `subjack`, `dnsx`, `nuclei`) |
| **Zone Transfer Misconfiguration** | A misconfigured DNS server can leak the **entire DNS zone** |
| **Cache Poisoning** | Injecting fake DNS entries to redirect traffic |
| **DNS Tunneling** | Used for **C2 communication** and **data exfiltration** |
| **Recon / Subdomain Enumeration** | Tools like `subfinder`, `amass`, `assetfinder` |

> 💡 **DNS isn't just Name → IP.** It's also a surface for **recon, takeover, and exfiltration.**

---

## 🛠 DNS Enumeration Commands (Cheat Sheet)

> DNS records live on the DNS server as zone files (`.txt` files).

```bash
# Get IPv4/IPv6 addresses for a domain
host google.com

# Get domain registrar info
whois google.com

# Find the nameservers hosting a domain
host -t ns google.com

# Find the mail servers used by a domain
host -t mx google.com

# Get IPv4/IPv6 addresses (also works on Windows)
nslookup google.com

# Interactive nslookup — query specific record types
nslookup
set type=CNAME
google.com
set type=AAAA
google.com

# dig — query any record type
dig google.com
dig google.com -t SOA
dig google.com -t TXT
```

> 🔑 **`nslookup`** and **`dig`** are the core **DNS query tools** used throughout recon.

| Command | Use Case |
|---|---|
| `host` | Quick A/AAAA/NS/MX lookups |
| `whois` | Domain registrar & ownership info |
| `nslookup` | Cross-platform DNS record lookup (interactive mode supports all record types) |
| `dig` | Detailed, flexible DNS queries — supports SOA, TXT, MX, and more |

---

## ✅ Key Takeaway

DNS resolution is more than just converting names to IPs — it's a **layered system** (local cache → recursive → root → TLD → authoritative) with real security implications: **misconfigured zones leak data, dangling CNAMEs get hijacked, and DNS itself can carry covert C2 traffic.** Understanding DNS deeply is a core recon skill for any pentester.

---

<div align="center">

📚 Solved as part of the **[TryHackMe](https://tryhackme.com/)** learning path
⭐ If this helped you, feel free to star the repo!

</div>
