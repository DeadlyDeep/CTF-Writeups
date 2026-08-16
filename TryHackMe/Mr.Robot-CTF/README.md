# Mr. Robot - CTF Writeup

## About the Challenge

**Mr. Robot** is a popular CTF (Capture The Flag) challenge inspired by the TV series of the same name. The challenge consists of finding three hidden flags scattered throughout a compromised website. The difficulty escalates through multiple stages requiring reconnaissance, web exploitation, privilege escalation, and hash cracking.

The challenge simulates a real penetration test scenario with multiple exploitation paths and intentionally misleading content (rabbit holes) to make the experience more challenging and authentic. All three flags must be captured in sequence: `key-1-of-3.txt`, `key-2-of-3.txt`, and `key-3-of-3.txt`.

---

## Reconnaissance & Information Gathering

### First Flag - robots.txt Enumeration
The first flag is easily accessible through directory enumeration and robots.txt analysis.

![Retrieved first flag (key-1-of-3.txt)](./images/image2.png)

### Website Source Code Analysis
Examined the HTML source code to look for hidden comments or credentials.

![HTML page source inspection](./images/image6.png)

### Finding Credentials in License Page
Discovered base64 encoded credentials hidden in the `/license` page of the website.

![Base64 decoding website with WordPress credentials](./images/image11.png)

The decoded credentials revealed:
- **Username:** elliot
- **Password:** ER28-0652

---

## Exploitation Phase 1: WordPress Access & Reverse Shell

### Logging into WordPress
Used the discovered credentials to access the WordPress admin panel.

![WordPress theme editor with reverse shell injection](./images/image12.png)

Successfully injected a PHP Pentestmonkey reverse shell into the **404.php** template file. The payload includes:
- Custom IP address (attacker VPN IP)
- Custom port for callback (9001 in this case)

### Setting Up Listener
Prepared a netcat listener to catch the incoming reverse shell connection.

![Netcat listener setup on port 9001](./images/image1.png)

### Triggering the Reverse Shell
Accessed the modified 404.php template to trigger the PHP code execution and establish the reverse shell connection.

![Incoming reverse shell connection received](./images/image4.png)

Successfully obtained shell access as the **www-data** user!

---

## Post-Exploitation & Lateral Movement

### Enumerating Home Directory
Navigated to `/home/robot` to search for the second flag and additional credentials.

![Exploring /home/robot directory](./images/image5.png)

Found:
- **key-2-of-3.txt** - (Permission denied, needs robot user privileges)
- **password.raw-md5** - MD5 hash of robot user's password

The MD5 hash discovered:
```
robot:c3fcd3d7192e4007dfb496cca67e13b
```

### Cracking the MD5 Hash
Used online hash cracking tools to reverse the MD5 hash and obtain the plaintext password.

The cracked password allowed SSH access as the robot user.

---

## Privilege Escalation to Robot User

### SSH Access as Robot
Connected via SSH using the cracked credentials to authenticate as the robot user.

Now with robot user privileges, we can read the second flag:

![Second flag (key-2-of-3.txt) retrieved](./images/image20.png)

---

## Privilege Escalation to Root

### Discovering SUID Nmap
While enumerating for privilege escalation vectors, discovered that the **nmap** binary has the SUID bit set - a critical misconfiguration!

```bash
find / -perm -4000 2>/dev/null
```

Nmap with SUID is highly unusual and exploitable.

### Nmap Interactive Mode Exploitation
Launched nmap in interactive mode and used the `!sh` command to spawn a shell with root privileges.

![Nmap interactive mode - executing !sh to gain root](./images/image3.png)

Successfully escalated to root user!

### Accessing Root Directory
Navigated to `/root` to retrieve the final flag.

![Discovering the final flag location](./images/image7.png)

### Third Flag Retrieved
Obtained the final flag:

```bash
root@kali:~# cat /root/key-3-of-3.txt
```

**All three flags successfully captured!** 🚩✅

---

## Summary

### Exploitation Chain
1. **Information Disclosure** - Base64 encoded credentials in `/license` page
2. **CMS Exploitation** - WordPress theme editor allows PHP code injection
3. **Remote Code Execution** - PHP reverse shell gives www-data shell access
4. **Credential Cracking** - MD5 hash reversed to obtain robot user password
5. **SSH Access** - Lateral movement to robot user account
6. **SUID Exploitation** - Nmap interactive mode allows root shell access

### Attack Timeline
```
Reconnaissance (robots.txt, source code)
    ↓
Credential Discovery (base64 decode)
    ↓
WordPress Access (theme editor RCE)
    ↓
Reverse Shell (www-data access)
    ↓
Lateral Movement (crack MD5, SSH as robot)
    ↓
Privilege Escalation (SUID nmap exploitation)
    ↓
Root Access & Flag Extraction
```

### Key Vulnerabilities Exploited
- **Hardcoded Credentials** - Base64 encoded credentials in web pages
- **Weak Password Hashing** - MD5 used for password storage (easily reversible)
- **CMS Misconfiguration** - Theme editor accessible to authenticated users
- **Insecure SUID Permissions** - Nmap binary with SUID and interactive mode
- **Lack of Input Validation** - PHP execution in theme templates

### Tools Used
- **gobuster** - Directory enumeration
- **netcat** - Reverse shell listener
- **ssh** - Lateral movement
- **nmap** - SUID exploitation vector
- **Base64 decoders** - Credential extraction
- **MD5 crack databases** - Password hash reversal

### Security Lessons
1. **Don't hardcode credentials** in code or configuration files, even if encoded
2. **Use strong hashing algorithms** (bcrypt, Argon2) instead of MD5
3. **Restrict CMS editor access** to trusted administrators only
4. **Audit SUID binaries** regularly and remove unnecessary ones
5. **Implement proper access controls** for sensitive files and directories
6. **Use principle of least privilege** - minimize user permissions

---

**Challenge Completed Successfully - All 3 Flags Captured!** 🎓

This writeup demonstrates how multiple seemingly unrelated misconfigurations can be chained together to achieve complete system compromise. Understanding and identifying these vulnerability patterns is essential for cybersecurity professionals.
