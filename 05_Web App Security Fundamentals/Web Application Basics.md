# 🌐 Web Application Basics — TryHackMe Writeup

![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen) ![Category](https://img.shields.io/badge/Category-Web%20Application%20Security-blue) ![Room](https://img.shields.io/badge/TryHackMe-Room-purple)

**Room Link:** [tryhackme.com/room/webapplicationbasics](https://tryhackme.com/room/webapplicationbasics)

---

## 📖 Overview

This room builds the foundational knowledge needed before diving into web application security testing — how web apps are structured, how URLs are built, and most importantly, how the HTTP protocol actually carries requests and responses between a browser and a server. Every web-based vulnerability class (SQLi, XSS, IDOR, open redirects) ultimately comes down to understanding and manipulating these fundamentals, which is exactly why this is treated as prerequisite knowledge.

> **Core Lesson:** You can't break what you don't understand. Every attack surface in a web app — the URL, the headers, the body, the response codes — is just HTTP doing exactly what it was designed to do, used in an unintended way.

---

## 🎯 What This Room Covers

| # | Topic | Key Focus |
|---|---|---|
| 1 | Web Application Overview | Front End vs Back End components |
| 2 | URL Anatomy | Scheme, host, port, path, query, fragment |
| 3 | HTTP Messages | Requests vs responses, message structure |
| 4 | HTTP Request Line & Methods | GET/POST/PUT/DELETE and friends |
| 5 | HTTP Request Headers & Body | Common headers, body encoding formats |
| 6 | HTTP Response Status Codes | 1xx–5xx categories |
| 7 | HTTP Response Headers & Body | Set-Cookie, Cache-Control, Location |
| 8 | Security Headers | CSP, HSTS, X-Content-Type-Options, Referrer-Policy |
| 9 | Practical Task | Making live GET/POST/DELETE requests |

---

## 1️⃣ Web Application Overview 🪐

The room uses a memorable planet analogy: a web app's **Front End** is the visible surface — what a user actually sees and touches in the browser — while the **Back End** is everything under the surface keeping it running.

**Front End components:**
- 🧬 **HTML** — the structural "DNA," defining what's displayed
- 🎨 **CSS** — visual styling: color, layout, typography
- 🧠 **JavaScript** — the "brain," enabling dynamic behavior and decision-making in-browser

**Back End components:**
- 🗄️ **Database** — stores and retrieves data (user info, preferences, content)
- 🏗️ **Infrastructure** — web/app servers, storage, networking that keep everything running
- 🛡️ **WAF (Web Application Firewall)** — an optional protective layer filtering malicious requests before they reach the server, much like an atmosphere shielding a planet's surface

---

## 2️⃣ Anatomy of a URL 🔗

A URL breaks down into several distinct, security-relevant components:

| Component | Example | Security Note |
|---|---|---|
| **Scheme** | `https://` | HTTPS encrypts the connection; HTTP does not |
| **User** | `user:pass@` | Rare today — embedding credentials in a URL is a real exposure risk |
| **Host/Domain** | `tryhackme.com` | Watch for **typosquatting** — near-identical fake domains used in phishing |
| **Port** | `:443` | Directs traffic to the right service; 80 = HTTP, 443 = HTTPS by default |
| **Path** | `/login` | Points to a specific resource; must be validated to prevent unauthorized access |
| **Query String** | `?id=5` | User-modifiable — a common **injection** entry point if not sanitized |
| **Fragment** | `#section` | Also user-modifiable; same injection caution applies |

---

## 3️⃣ HTTP Messages — The Client/Server Conversation 💬

Every interaction with a web app boils down to two message types:
- 📤 **HTTP Request** — sent by the client to trigger an action
- 📥 **HTTP Response** — sent back by the server

Both share a consistent structure:

| Part | Purpose |
|---|---|
| **Start Line** | Identifies the message type and how it should be handled |
| **Headers** | Key-value metadata about the request/response |
| **Empty Line** | Separates headers from the body — critical for correct parsing |
| **Body** | The actual payload — form data, JSON, HTML, etc. |

---

## 4️⃣ HTTP Request Line & Methods 📨

**Format:** `METHOD /path HTTP/version`

| Method | Purpose | Security Consideration |
|---|---|---|
| **GET** | Fetch data | Never put sensitive data (tokens, passwords) in a GET — it can leak in plaintext, logs, or browser history |
| **POST** | Create/submit data | Always validate & sanitize input to prevent SQLi/XSS |
| **PUT** | Replace/update a resource | Requires proper authorization checks |
| **DELETE** | Remove a resource | Same — must verify the requester is authorized |
| **PATCH** | Partial update | Validate to avoid data inconsistency |
| **HEAD** | Like GET, headers only | Useful for checking metadata without pulling full content |
| **OPTIONS** | Lists supported methods for a resource | Should be disabled if not explicitly needed |
| **TRACE** | Debug/echo method | Frequently disabled — a known security risk if left on |
| **CONNECT** | Establishes tunnels (e.g., for HTTPS) | Critical for encrypted communication |

### 📜 HTTP Version Timeline

| Version | Year | Notable Addition |
|---|---|---|
| HTTP/0.9 | 1991 | GET only — the bare minimum |
| HTTP/1.0 | 1996 | Headers, better content support |
| HTTP/1.1 | 1997 | Persistent connections, chunked encoding — **still the most widely used** |
| HTTP/2 | 2015 | Multiplexing, header compression |
| HTTP/3 | 2022 | Runs over QUIC for faster, more secure connections |

---

## 5️⃣ Request Headers & Body Formats 📦

**Common request headers:**

| Header | Example | Purpose |
|---|---|---|
| `Host` | `Host: tryhackme.com` | Target server for the request |
| `User-Agent` | `Mozilla/5.0` | Identifies the client software |
| `Referer` | `https://google.com/` | Where the request originated from |
| `Cookie` | `session=abc123` | Client-stored data sent back to the server |
| `Content-Type` | `application/json` | Format of the data in the body |

**Body encoding formats:**

- 🔗 **URL Encoded** (`application/x-www-form-urlencoded`) — `key=value&key2=value2`, special characters percent-encoded
- 📎 **Form Data** (`multipart/form-data`) — used for file/binary uploads, separated by boundary strings
- 🧾 **JSON** (`application/json`) — modern API standard, `{ "key": "value" }`
- 🏷️ **XML** (`application/xml`) — nested tag-based structure, `<user><name>...</name></user>`

---

## 6️⃣ HTTP Response: Status Line & Status Codes 🚦

Every response starts with a **Status Line**: HTTP version + status code + reason phrase.

| Range | Category | Meaning |
|---|---|---|
| 🔵 100–199 | Informational | "Keep going," partial request received |
| 🟢 200–299 | Success | Request processed correctly |
| 🟡 300–399 | Redirection | Resource moved — check the `Location` header |
| 🟠 400–499 | Client Error | Problem with the request itself (bad URL, missing auth) |
| 🔴 500–599 | Server Error | Something broke server-side, not the client's fault |

**Most common codes:**
- ✅ `200 OK` — success
- 🔀 `301 Moved Permanently` — resource relocated
- ❓ `404 Not Found` — resource doesn't exist at that path
- 💥 `500 Internal Server Error` — server-side failure

---

## 7️⃣ Response Headers & Body 📬

**Key response headers:**

| Header | Example | Notes |
|---|---|---|
| `Date` | `Fri, 23 Aug 2024 10:43:21 GMT` | When the response was generated |
| `Content-Type` | `text/html; charset=utf-8` | Format + character set of the content |
| `Server` | `nginx` | ⚠️ Reveals server software — often removed to reduce attacker recon value |
| `Set-Cookie` | `sessionId=38af1337es7a8` | Should use `HttpOnly` (blocks JS access) + `Secure` (HTTPS only) flags |
| `Cache-Control` | `max-age=600` | Controls caching duration; `no-cache` protects sensitive data |
| `Location` | `/index.html` | Used in redirects — unsanitized values risk **open redirect vulnerabilities** |

**Response Body:** the actual content (HTML/JSON/images). User-generated content going into the body must always be sanitized/escaped to prevent **XSS**.

---

## 8️⃣ Security Headers — Hardening the Response 🛡️

A quick way to audit any site's security headers is a tool like [securityheaders.io](https://securityheaders.io/). The room covers four key ones:

### 🔒 Content-Security-Policy (CSP)
Restricts which domains can serve scripts, styles, and other resources — mitigating XSS from injected third-party content.
```
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.tryhackme.com; style-src 'self'
```
- `default-src` — fallback policy
- `script-src` — allowed script sources
- `style-src` — allowed stylesheet sources
- `'self'` — a special keyword meaning "same origin only"

### 🔐 Strict-Transport-Security (HSTS)
Forces browsers to always connect over HTTPS.
```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```
- `max-age` — how long (seconds) the policy is enforced
- `includeSubDomains` — extends the rule to all subdomains
- `preload` — allows inclusion in browser HSTS preload lists, enforced even before first visit

### 🧪 X-Content-Type-Options
Stops browsers from guessing (MIME-sniffing) a resource's type.
```
X-Content-Type-Options: nosniff
```

### 🔗 Referrer-Policy
Controls how much referrer info is leaked when navigating away from a page.

| Directive | Behavior |
|---|---|
| `no-referrer` | Sends nothing |
| `same-origin` | Only sent within the same site |
| `strict-origin` | Only sent if protocol stays the same (HTTPS→HTTPS) |
| `strict-origin-when-cross-origin` | Full path for same-origin, origin-only for cross-origin |

---

## 9️⃣ Practical Task — Making Real Requests 🧪

The room provides a live emulator to practice against, requiring:
- A **GET** request to `/api/users`
- A **POST** request to `/api/user/2` updating a user's country field
- A **DELETE** request to `/api/user/1` removing that user

This ties every earlier concept together — method selection, path targeting, and body formatting — into hands-on practice with real request/response cycles.

---

## 🧠 Key Lessons

- 🏗️ **Front end vs back end** — knowing where a component lives helps scope where a vulnerability could realistically exist.
- 🔗 **Every URL component is a potential input** — host, path, query, and fragment can all be attacker-controlled to varying degrees.
- 📨 **Method choice has security implications** — GET leaks data in logs/history; POST/PUT/DELETE all need proper authorization checks.
- 🚦 **Status codes tell a story** — they're often the fastest way to fingerprint how an app is behaving (or misbehaving).
- 🛡️ **Security headers are cheap, high-value hardening** — CSP, HSTS, X-Content-Type-Options, and Referrer-Policy each close off a specific class of attack with minimal server-side effort.

---

## 🛠️ Concepts & Tools Covered

`HTTP/HTTPS` `URL Structure` `HTTP Methods (GET/POST/PUT/DELETE/PATCH)` `Status Codes` `Request/Response Headers` `Cookies (HttpOnly/Secure)` `CSP` `HSTS` `X-Content-Type-Options` `Referrer-Policy` `securityheaders.io`

---

## ✅ Status: Completed

*Room completed independently on TryHackMe. Writeup reflects my own notes and understanding of the material — specific flag values are intentionally omitted.*
