# 🔴 Offensive Security Intro — TryHackMe Writeup

![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen) ![Category](https://img.shields.io/badge/Category-Offensive%20Security-red) ![Room](https://img.shields.io/badge/TryHackMe-Room-purple)

**Room Link:** [tryhackme.com/room/offensivesecurityintro](https://tryhackme.com/room/offensivesecurityintro)

---

## 📖 Overview

This room introduces the mindset behind **Offensive Security** — the practice of thinking and acting like an attacker in order to uncover weaknesses before real adversaries do. The hands-on exercise involved attacking a simulated bank web application ("FakeBank") in a safe, legal environment to understand how ethical hackers identify and exploit vulnerabilities.

> **Core Principle:** *"To outsmart a hacker, you need to think like one."*

---

## 🎯 Objective

Simulate a real-world attack scenario on a fake banking application to:
- Discover hidden/unlisted web pages using directory brute-forcing
- Identify a vulnerable, unauthenticated admin function
- Demonstrate real-world impact by manipulating account data

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| **Gobuster** | Directory/content discovery — brute-forces URLs against a wordlist to find hidden pages |
| **Web Browser** | Interacting with the discovered endpoints |
| **Terminal** | Executing reconnaissance commands |

---

## 🔍 Methodology

### Step 1 — Reconnaissance with Gobuster
Many web applications have hidden admin panels or functional pages that aren't linked anywhere on the public site. These are often discoverable simply because they weren't properly restricted.

Using Gobuster, I ran a directory brute-force scan against the target:

```bash
gobuster -u http://fakebank.thm -w wordlist.txt dir
```

**Flags used:**
- `-u` → target URL
- `-w` → wordlist to iterate through for potential directory/page names
- `dir` → mode for directory brute-forcing

### Step 2 — Analyzing the Results
The scan returned a list of valid paths along with their HTTP status codes. A `200 OK` response confirmed the page existed and was accessible without authentication:

```
/images         (Status: 301)
/bank-transfer  (Status: 200)
```

The `/bank-transfer` endpoint immediately stood out — a sensitive banking function exposed with **no access control**.

### Step 3 — Exploiting the Exposed Endpoint
Navigating directly to `/bank-transfer` revealed a functional money-transfer form — accessible to anyone, without login or authorization checks. This is a textbook example of **broken access control**, one of the most common and dangerous web vulnerabilities (OWASP Top 10).

Using this page, I performed a proof-of-concept transfer between two accounts, confirming that an attacker could move funds from any account without authorization.

---

## 💡 Key Takeaway

This lab is a practical demonstration of why **unauthenticated, unlisted pages are still a real attack surface**. Even if a page isn't linked in the site's navigation, it can still be discovered and exploited if it's reachable via a direct URL. This reinforces the importance of:
- Enforcing **authentication and authorization** on every sensitive endpoint — not just visible ones
- Never relying on **"security through obscurity"**
- Regularly auditing applications with tools like Gobuster from an attacker's perspective

---

## 🧠 Skills Practiced

`Reconnaissance` `Directory Brute-Forcing` `Broken Access Control` `Web Application Security` `Ethical Hacking Methodology`

---

## ✅ Status: Completed

*This writeup documents my personal learning process. TryHackMe answers/flags are intentionally omitted — this is a walkthrough of methodology, not a copy-paste solution.*
