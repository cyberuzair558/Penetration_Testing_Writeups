# 🛡️ OWASP Top 10 2025: IAAA Failures — TryHackMe Writeup

![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge&logo=tryhackme)
![Category](https://img.shields.io/badge/Category-Web%20App%20Security-blue?style=for-the-badge)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)

> **Room:** [OWASP Top 10 2025: IAAA Failures](https://tryhackme.com/room/owasptopten2025one) — TryHackMe
> **Author of writeup:** *Your Name Here*
> **Purpose:** Deep-diving into 3 of the OWASP Top 10 2025 categories — Broken Access Control (A01), Authentication Failures (A07), and Logging & Alerting Failures (A09) — through the lens of the IAAA security model, with hands-on practical challenges for each.

---

## 📑 Table of Contents

- [Introduction](#-introduction)
- [What is IAAA?](#-what-is-iaaa)
- [A01: Broken Access Control](#-a01-broken-access-control)
- [A07: Authentication Failures](#-a07-authentication-failures)
- [A09: Logging & Alerting Failures](#-a09-logging--alerting-failures)
- [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)
- [Key Takeaways](#-key-takeaways)

---

## 📝 Introduction

This room is the first in a series that breaks down the **OWASP Top 10 2025** into digestible, practical modules. Instead of covering all 10 categories at once, this room zooms in on the **three vulnerabilities that stem from failures in the IAAA model** — how an application handles Identity, Authentication, Authorisation, and Accountability.

Each category comes with its own static practice site, so the theory is immediately reinforced with a hands-on challenge.

This writeup documents my notes while completing the **OWASP Top 10 2025: IAAA Failures** room on TryHackMe.

---

## 🧩 What is IAAA?

**IAAA** is a mental model for understanding how users and their actions are verified within an application. It has 4 sequential layers — and importantly, **you can't skip a layer**: if a lower layer fails, everything built on top of it is compromised.

| Layer | Meaning |
|---|---|
| **Identity** | The unique account (user ID/email) that represents a person or service |
| **Authentication** | Proving that identity — passwords, OTP, passkeys |
| **Authorisation** | What that identity is *allowed* to do |
| **Accountability** | Recording and alerting on who did what, when, and from where |

> 📌 The three OWASP categories covered in this room each represent a **breakdown at a different IAAA layer**: A01 breaks Authorisation, A07 breaks Authentication, and A09 breaks Accountability.

---

## 🔓 A01: Broken Access Control

**Core issue:** the server doesn't properly enforce **who can access what** on every single request — it trusts the client too much.

### 🔹 IDOR (Insecure Direct Object Reference)

The most common manifestation of Broken Access Control. If changing an ID in a request (e.g. `?id=7` → `?id=6`) lets you view or edit **someone else's data**, access control is broken — the server never checked whether the requester actually *owns* that resource.

### 🔹 Two flavours of privilege escalation

| Type | What it means |
|---|---|
| **Horizontal privilege escalation** | Same role/privilege level, but accessing **another user's** data (e.g. Ali viewing Sara's notes) |
| **Vertical privilege escalation** | Jumping to a **higher privilege level** — e.g. a normal user reaching admin-only actions |

### 🧪 Practical challenge

The attached static site displays account information tied to an `accountID` value in the URL. By manipulating this parameter, it's possible to browse through accounts that don't belong to you — including finding a specific account holding **over $1 million**, along with a hidden note left on that profile.

### 🎯 What the correct fix looks like

The server should never rely solely on "is this session logged in?" — it must also ask, on **every** request: *"does this specific resource actually belong to this specific session?"* This means checking resource ownership against the database (e.g. comparing the session's user ID against the `owner_id` column of the requested record) before returning any data.

### 📚 Rooms to go deeper afterwards
- [Broken Access Control](https://tryhackme.com/room/owaspbrokenaccesscontrol)
- [Insecure Direct Object References](https://tryhackme.com/room/idor)

---

## 🔑 A07: Authentication Failures

**Core issue:** the application can't reliably verify or bind a user's identity. Common causes:

| Weakness | What it enables |
|---|---|
| **Username enumeration** | Different error messages for "user not found" vs "wrong password" let an attacker build a list of valid usernames |
| **Weak/guessable passwords, no lockout/rate limits** | Enables brute-force attacks — unlimited attempts, no complexity requirements |
| **Logic flaws in login/registration flow** | Inconsistent handling somewhere in the signup → verification → login pipeline |
| **Insecure session or cookie handling** | Predictable or stealable session identifiers let an attacker bind a session to the wrong account |

### 🧪 Practical challenge — case-sensitivity exploit

The room demonstrates a classic **logic flaw**: the target application's real admin account is named `admin`. By registering a new account using a **different letter-casing of the same username** (e.g. mixing upper/lowercase), the registration check treats it as a brand-new, distinct username and allows it — but elsewhere in the application (login/session/display logic), the comparison is handled inconsistently, and the attacker ends up landing in the **real admin's dashboard**.

**Why this works:** the application applies two different standards to the same value — case-sensitive in one place (registration uniqueness check), case-insensitive in another (account matching/display) — and that inconsistency becomes the security hole.

### 📚 Rooms to go deeper afterwards
- [Authentication Bypass Room](https://tryhackme.com/room/authenticationbypass)
- [Multi-Factor Authentication](https://tryhackme.com/room/multifactorauthentications)
- [Authentication Module](https://tryhackme.com/module/authentication)

---

## 📋 A09: Logging & Alerting Failures

**Core issue:** when applications don't record or alert on security-relevant events, defenders can't detect or investigate attacks. Good logging is what makes **Accountability** (the 4th IAAA layer) possible.

### 🔹 What failures look like in practice
- Missing authentication events (no record of failed/successful logins)
- Vague, unhelpful error logs
- No alerting on brute-force patterns or privilege changes
- Short log retention periods
- Logs stored somewhere an attacker could tamper with them

### 🧪 Practical challenge — reading an attack from the logs

The attached static site presents a **log viewer dashboard** showing a stream of HTTP requests hitting a login endpoint. The investigation task is to trace a **brute-force attack** from start to finish by reading through the logs:

1. **Identify the attacker's IP** — the log entries repeatedly show `401 Unauthorized` responses from the same IP address hitting `/login` with different password guesses (`admin123`, `password`, `12345`, `letmein`...) — a textbook brute-force pattern.
2. **Find the successful breach** — scrolling further down the log stream reveals the point where a `401` finally flips to a **`200` success**, meaning one of the guessed credentials worked and the attacker gained account access.
3. **Identify what the attacker did next** — immediately after the successful login, the following log entries show the attacker interacting with the account — the exact endpoint they accessed (e.g. an account action like a transfer or settings change) is the final piece of the investigation.

### 🧠 The bigger lesson

Even *with* full logs, spotting this attack required manually reading a chronological stream of raw log lines. In a real system, this same pattern — repeated `401`s from one IP followed by a `200` — is exactly what an **automated alerting rule** should catch and flag *before* the attacker gets that far. This is the difference between logging (recording data) and alerting (acting on it in real time).

### 📚 Room to go deeper afterwards
- [Logging for Accountability](https://tryhackme.com/room/loggingforaccountability)

---

## ⚡ Quick Revision Cheat Sheet

```
IAAA MODEL                    →  MEANING
──────────────────────────────────────────────────────────────
Identity                       →  Who the account claims to be
Authentication                 →  Proving that identity
Authorisation                  →  What that identity can do
Accountability                 →  Recording/alerting on actions

OWASP CATEGORY                 →  BROKEN IAAA LAYER  →  CORE FLAW
──────────────────────────────────────────────────────────────
A01 Broken Access Control      →  Authorisation      →  Server trusts client-supplied IDs, no ownership check
A07 Authentication Failures    →  Authentication      →  Inconsistent/weak identity verification logic
A09 Logging & Alerting Failures→  Accountability      →  No record/alert of malicious activity

PRACTICAL TASK                 →  TECHNIQUE USED
──────────────────────────────────────────────────────────────
A01 — accountID manipulation   →  Changing an ID in the URL (IDOR) to view another account
A07 — aDmiN registration       →  Case-sensitivity logic flaw to hijack the real "admin" identity
A09 — log timeline analysis    →  Tracing 401s → successful 200 → post-breach attacker action
```

---

## 🎯 Key Takeaways

- **IAAA is sequential** — Identity → Authentication → Authorisation → Accountability. A failure in an earlier layer undermines everything built on top of it.
- **A01 Broken Access Control:** enforce server-side ownership checks on **every** request — never trust a client-supplied ID at face value. Distinguish between horizontal (same-level, other user) and vertical (privilege jump) escalation.
- **A07 Authentication Failures:** enforce consistent, canonical identity comparisons (e.g. normalize case before checking uniqueness), rate-limit/lock out brute-force attempts, and rotate sessions on password/privilege changes.
- **A09 Logging & Alerting Failures:** logging the full auth lifecycle (failures, successes, role changes, admin actions) is only half the job — logs need to be centralised, retained, and actively **alerted on** (e.g. brute-force bursts) rather than just passively stored.
- Real-world defenders don't just need logs to exist — they need logs that are **complete enough and monitored closely enough** to catch an attack while it's happening, not just after the fact.

---

<p align="center">
  <i>📌 Part of my TryHackMe learning journey — see <b>03-Intro-to-Web-Application-Security.md</b> for the foundational OWASP intro, and stay tuned for <b>Room 2: Application Design Flaws</b> in this same module!</i>
</p>
