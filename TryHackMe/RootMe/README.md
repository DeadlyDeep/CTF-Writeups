# Root.me - CTF Writeup
[Visit CTF on tryhackme](https://tryhackme.com/room/rrootme)
## Reconnaissance

### Port Scanning
Performed an Nmap scan and identified two open ports: SSH and HTTP.

![Nmap scan results showing SSH and HTTP ports](./images/image5.png)

### DNS Configuration
Added the target URL address to the `/etc/hosts` file for proper DNS resolution.

![Adding URL to /etc/hosts](./images/image1.png)

### Website Exploration
The website homepage looks fairly simple with minimal content.

![Website homepage](./images/image12.png)

Checked the page source to look for any hidden clues, but it only contained an empty comment.

![Empty page source with just a comment](./images/image3.png)

### Directory Fuzzing
Performed directory fuzzing using **gobuster** with the `common.txt` wordlist and discovered several directories.

![Gobuster directory enumeration results](./images/image11.png)

### File Upload Functionality
Discovered the `/panel` directory, which contained a file upload page.

![File upload interface found in /panel](./images/image7.png)

## Exploitation

### Initial Access - PHP Reverse Shell Upload
Attempted to upload a PHP reverse shell to gain remote code execution. I used PHP Pentestmonkey's reverse shell from [revshells.com](http://revshells.com).

**Important:** Remember to modify the IP and port in the payload to match your VPN IP and an available port on your machine.

The initial upload was named `payload.php`, but it failed due to extension filtering.

![PHP file upload blocked - .php extension not allowed](./images/image14.png)

### Bypassing File Upload Restrictions
The website blocked `.php` file extensions. I bypassed this restriction by renaming the file to `.phtml` using the `mv` command.

![Renaming payload.php to payload.phtml](./images/image8.png)

The file was successfully uploaded with the `.phtml` extension.

![Successful .phtml file upload](./images/image17.png)

### Establishing Reverse Shell
Set up a netcat listener on the port specified in the reverse shell payload.

![Netcat listener setup](./images/image10.png)

Accessed the uploaded shell by navigating to `http://$ip/uploads/payload.phtml`, which triggered the reverse shell connection.

![Reverse shell connection established](./images/image16.png)

### Shell Stabilization
Stabilized the shell using Python to get a proper interactive terminal.

![Shell stabilization using Python](./images/image13.png)

## Privilege Escalation

### Finding the User Flag
Changed to the `/home` directory, which is the most common location for user flags. Found three subdirectories (`/rootme`, `/test`, `/ubuntu`) but they were empty.

Used the `find` command to locate the `user.txt` file and successfully retrieved it.

![Locating and reading user.txt flag](./images/image9.png)

### SUID Binary Analysis
Checked for SUID binaries to find privilege escalation vectors.

![SUID binaries scan showing python2.7 as unusual](./images/image2.png)

**Key Finding:** The `python2.7` binary has SUID permissions, which is unusual and exploitable.

### Exploitation via GTFObins
Searched GTFObins for Python 2.7 privilege escalation techniques and found an exploit.

![GTFObins python2.7 privilege escalation method](./images/image4.png)

### Becoming Root
Executed the privilege escalation command in the shell:

![Running the Python privilege escalation command](./images/image15.png)

Successfully gained root access!

### Root Flag
Changed to the `/root` directory and retrieved the root flag from `root.txt`.

![Root flag captured - CTF complete](./images/image6.png)

## Summary

This CTF demonstrated several common vulnerabilities:

1. **File Upload Vulnerabilities** - Simple extension-based filtering can be bypassed with alternative extensions (`.phtml`, `.phar`, etc.)
2. **Weak SUID Permissions** - Allowing high-level interpreters like Python to run with SUID is a major security risk
3. **Directory Enumeration** - Standard directory names made enumeration straightforward
4. **Lack of Input Validation** - No proper validation on uploaded files

**Key Tools Used:**
- `nmap` - Port scanning
- `gobuster` - Directory enumeration
- `netcat` - Reverse shell listener
- `python` - Shell stabilization and privilege escalation
- `find` - File search

**Exploited Vulnerabilities:**
- Unrestricted file upload with weak extension filtering
- SUID binary with privilege escalation capability (Python)
