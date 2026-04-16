# 🧠 CTF Writeup – SweetRice CMS Exploitation & Privilege Escalation

## 📌 Overview
This machine involved web enumeration, credential extraction from a backup file, gaining a reverse shell via CMS upload functionality, and privilege escalation through a misconfigured script.

---

## 🔍 Reconnaissance

### Nmap Scan
`nmap -sV -sC -p- 10.112.155.62`

Results:

```PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```

Default Apache page observed.

[[IMAGE]](https://github.com/hyp3r-1on/ctf-writeups/blob/6f34425bb0c36ec31bb3eb3a2eb626886e4fa830/tryhackme/assets/Apache2%20Defualt.png)

## 📂 Directory Enumeration
### Initial Scan
`gobuster dir -u http://10.112.155.62 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt`

Found /content/

[IMAGE]

### Deeper Scan
`gobuster dir -u http://10.112.155.62/content -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt`

### Directories:

```/content/images
/content/js
/content/inc
/content/as
/content/_themes
/content/attachment
```

## 🧩 CMS Discovery

The application is running **SweetRice CMS**.

Inside `/content/inc/`, a backup directory was found:

[IMAGE]

## 🔐 Credential Extraction

Downloaded SQL backup revealed:

```"admin";s:7:"manager";s:6:"passwd";s:32:"42f749ade7f9e195bf475f37a44cafcb";
```
- Username: `manager`
- Password hash: `42f749ade7f9e195bf475f37a44cafcb`

Cracked password:

`Password123`
## 🔑 Admin Access

Login page:

`http://10.112.155.62/content/as/`

### Credentials:

`manager : Password123`

Access to dashboard achieved.

[IMAGE]

## 💥 Reverse Shell
### Upload Payload
  - Navigated to **Media Center**
  - Uploaded ZIP containing PHP reverse shell
  - Enabled extraction

### Listener
`nc -nvlp 1234`

Triggered payload → shell as `www-data`

[IMAGE]

## 🛠️ Shell Stabilization

- `python3 -c 'import pty; pty.spawn("/bin/bash")'`
- **Ctrl + Z** to take shell background
- `stty raw -echo; fg`
- `export TERM=xterm`


## 👤 User Access
  - Navigated to `/home`
  - Found user `itguy`
  - Retrieved user flag

[IMAGE]

## 🔼 Privilege Escalation
### Backup Script

Found:
`backup.pl`

This executes:
`/etc/copy.sh`

### Weak Permissions
`ls -l /etc/copy.sh`
- File is writable

### Exploit

Modified `/etc/copy.sh` to include malicious command.

### Sudo Permissions
`sudo -l`
- Allowed to run backup.pl as root

### Root Shell
`sudo /usr/bin/perl /path/to/backup.pl`
→ Executes modified script → root shell

[IMAGE]

## 🏁 Root Flag

Located in:
`/root`

[IMAGE]

## 🧾 Summary
Step	Description
Recon	Nmap scan
Enum	Found /content/
Exploit	SQL backup leak
Access	CMS login
Shell	File upload RCE
Privesc	Writable script + sudo
Result	Root access
