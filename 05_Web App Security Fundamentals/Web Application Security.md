# 🌐 Intro to Web Application Security — TryHackMe Writeup

![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge&logo=tryhackme)
![Category](https://img.shields.io/badge/Category-Web%20App%20Security-blue?style=for-the-badge)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)

> **Room:** [Intro to Web Application Security](https://tryhackme.com/room/introwebapplicationsecurity) — TryHackMe
> **Author of writeup:** *Your Name Here*
> **Purpose:** Understanding the fundamentals of web applications, the main stages where attackers target them, and a first look at OWASP Top 10 vulnerability categories — with a hands-on practical demonstrating Broken Access Control (IDOR).

---

## 📑 Table of Contents

- [Introduction](#-introduction)
- [What is a Web Application?](#-what-is-a-web-application)
- [Main Attack Vectors](#-main-attack-vectors)
- [OWASP Top 10 — Vulnerabilities Covered in This Room](#-owasp-top-10--vulnerabilities-covered-in-this-room)
- [Practical: Broken Access Control (IDOR) Walkthrough](#-practical-broken-access-control-idor-walkthrough)
- [Understanding "Revert" in This Task](#-understanding-revert-in-this-task)
- [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)
- [Key Takeaways](#-key-takeaways)

---

## 📝 Introduction

Before diving into specific vulnerability classes like the **OWASP Top 10**, it's important to understand *how* a web application works and *where* an attacker typically tries to break in. This room lays that foundation — it explains what a web application is, walks through the stages of a typical user journey (login → search → checkout), and maps each stage to a category of risk from the OWASP Top 10.

This writeup documents my notes while completing the **Intro to Web Application Security** room on TryHackMe.

---

## 🧩 What is a Web Application?

A **web application** is a program that runs inside a browser rather than being installed directly on your computer. The actual processing and storage happen on a **remote server**, which typically has far more computing power and storage capacity than an individual's own machine, and serves many users at once.

- **Access requirement:** all you need is a **web browser**.
- Examples: online shopping sites, webmail, social media platforms, banking portals.

A typical example — an online shop — involves these steps:

1. **Log in** to the account.
2. **Search** for an item.
3. **Add** the item to the cart.
4. **Provide address/shipping** details.
5. **Provide payment** details.

Each of these steps is a potential point of attack.

---

## 🎯 Main Attack Vectors

| Stage | Possible Attack |
|---|---|
| **Login** | Attacker tries to guess or brute-force credentials to gain unauthorized access. |
| **Search / Input fields** | Attacker manipulates input so the application returns data it shouldn't, or executes commands it shouldn't (injection-style attacks). |
| **Payment / Sensitive Data** | Attacker intercepts data sent in cleartext or protected by weak encryption. |

This is exactly why the **OWASP Top 10** exists — to catalogue the most common and critical risks across these attack surfaces.

---

## 🛡️ OWASP Top 10 — Vulnerabilities Covered in This Room

The **Open Web Application Security Project (OWASP)** is a non-profit foundation that publishes the **OWASP Top 10** — the industry-standard list of the most critical web application security risks. This room introduces three of them:

### 🔹 1. Broken Access Control

Access control makes sure a user can only reach the resources tied to their own role or account.

- Failing to apply the **principle of least privilege** — giving users more permissions than they need (e.g. a customer being able to *change* prices instead of just viewing them).
- Being able to view or modify **someone else's data** just by changing an identifier in a request (this is the basis of **IDOR** — Insecure Direct Object Reference, demonstrated practically later in this room).
- Being able to browse **authenticated-only pages** without ever logging in.

### 🔹 2. Injection

An injection vulnerability happens when user-supplied input is **not properly validated or sanitized**, allowing an attacker to insert malicious code/commands into a field that the application then executes or misuses — leading to data manipulation or full system compromise.

### 🔹 3. Identification and Authentication Failure

- **Identification** = uniquely identifying a user (e.g. a username).
- **Authentication** = proving that a user really is who they claim to be (e.g. a password check).

Common weaknesses in this category:

| Weakness | Risk |
|---|---|
| No limit on login attempts | Enables **brute-force** attacks |
| Weak password policy allowed | Passwords are easy to guess |
| Passwords/credentials sent or stored in **cleartext** | Anyone intercepting traffic or reading the storage can read the password directly |

> 📌 Note: OWASP Top 10 is **not exhaustive** — it's a snapshot of the most common/critical risks, not a complete list of every possible vulnerability.

---

## 🧪 Practical: Broken Access Control (IDOR) Walkthrough

The room includes a hands-on task built around a simple **note-taking web app**, used to demonstrate an **IDOR (Insecure Direct Object Reference)** vulnerability — a sub-category of Broken Access Control.

**Scenario:**

- Deploy the target machine and browse to `http://MACHINE_IP`.
- Log in with the provided low-privilege credentials.
- The app lets a logged-in user view their own note via a URL containing a note identifier, e.g.:

```
http://MACHINE_IP/note?noteid=1
```

- Because the server does **not check whether the requesting user actually owns that note ID**, simply changing the number in the URL (e.g. `noteid=0`) lets you view **another user's private note** — this is IDOR in action.

**Why this matters:** the vulnerability isn't in the login system at all — authentication worked fine. The flaw is that **after** login, the app trusts a user-controllable identifier without verifying ownership. This is exactly the kind of "Broken Access Control" weakness described above.

---

## 🔁 Understanding "Revert" in This Task

One of the room's questions asks you to:

> *"Check the other users to discover which user account was used to make the malicious changes and revert them. After reverting the changes, what is the flag that you have received?"*

**What "revert" means here:**

- **Revert = to undo a change and restore something back to its previous/original state.**
- In this context, an attacker (using a compromised or malicious account) has altered some data inside the application — for example, changed the content of a note, a price, or a setting.
- Your job as the investigator is to:
  1. **Identify** which user account performed the unauthorized/malicious edit (often by browsing other users' notes/records via the IDOR flaw itself, or by checking version/edit history).
  2. **Revert** — i.e., **restore the original value** — by editing the content back to what it should be, or by using an "undo"/"restore version" feature if the app provides one.
  3. Once the malicious change has been successfully reverted, the application reveals the **flag** as confirmation that the correct state has been restored.

**Why "revert" matters as a concept (not just for this room):**

- In real-world incident response, **reverting** malicious or unauthorized changes is a core remediation step after detecting a compromise — you don't just find the bad actor, you also restore the system/data to a known-good state.
- It's the same idea used in version control systems (e.g. `git revert`) — undoing a specific change without necessarily deleting the history of that it happened.

---

## ⚡ Quick Revision Cheat Sheet

```
WEB APP BASICS               →  DETAIL
──────────────────────────────────────────────────────────────
Access requirement            →  Just a web browser
Runs on                       →  Remote server (not your PC)
Typical user flow             →  Login → Search → Cart → Payment

ATTACK VECTOR                 →  RISK
──────────────────────────────────────────────────────────────
Login page                    →  Brute-force / credential guessing
Search / input fields         →  Injection-style manipulation
Payment / sensitive data      →  Cleartext or weak encryption exposure

OWASP CATEGORY (this room)    →  CORE IDEA
──────────────────────────────────────────────────────────────
Broken Access Control         →  User reaches data/actions beyond their role (e.g. IDOR)
Injection                     →  Unsanitized input executed/misused by the app
Identification & Auth Failure →  Weak/no brute-force protection, weak passwords, cleartext creds

PRACTICAL TASK                →  TAKEAWAY
──────────────────────────────────────────────────────────────
IDOR demo (noteid=X)          →  Changing an ID in the URL exposes other users' data
"Revert" question             →  Find the malicious change → restore original state → get flag
```

---

## 🎯 Key Takeaways

- A **web application** runs on a remote server and is accessed purely through a browser — no installation needed.
- Every step of a typical user journey (**login, search, payment**) is a potential attack surface.
- **OWASP Top 10** is the industry-standard (but not exhaustive) list of the most critical web app security risks.
- **Broken Access Control** — including **IDOR** — happens when the app fails to verify that a user actually owns the resource they're requesting, even though they're properly logged in.
- **Injection** flaws stem from missing input validation/sanitization.
- **Identification and Authentication Failure** covers weak login protections: no brute-force lockout, weak passwords, cleartext credential storage/transmission.
- **"Revert"** = undoing/restoring a malicious or unwanted change back to its original, legitimate state — a key step in both this room's practical task and real-world incident response.

---

<p align="center">
  <i>📌 Part of my TryHackMe learning journey — see <b>02-Nmap-Basic-Port-Scanning.md</b> for Nmap scanning fundamentals, and stay tuned for the full OWASP Top 10 room writeup next!</i>
</p>
