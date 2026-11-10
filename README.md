# Week 2: Footprinting & Network Scanning Project
PENETRATION TESTING REPORT
FOOTPRINTING & NETWORK SCANNING PHASES
 | W2-PM-FINAL | CYBERSECURITY | NETWORKWALKS
Field
Details
Pentester Name
ALABI ISREAL  
Program/Batch
B084 – Networkwalks
Date
10 November 2026
Modules completed
W2-PM1 (Multiple Kali Tools)
W2-PM5 (Zenmap Scanning)
Client/Target
Networkwalks – networkwalks.com (written permission secured)
Permission secured from client?
Yes
Phases covered
Phase 1: Reconnaissance  Footprinting; Phase 2: Scanning & Network Discovery, Phase 3-5: In Progress

1. Liability Disclaimer
I have performed these activities only on the systems and devices where I had secured written permission, or on devices and systems that I own myself. All these materials are for education and research purposes only. Do not use anything from here to break the law. The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries unauthorised access is a crime even when nothing is damaged.
2. Introduction
This report covers footprinting the networkwalks.com domain using multiple Kali Linux tools (W2-PM1) and scanning the same target with Zenmap (W2-PM5). One module covers the footprinting phase and the other covers the scanning phase, so together they show how an attacker moves from gathering public information to mapping the services a host exposes. It is the Week 2 part of my ongoing internship program at Networkwalks.
All commands were run in Kali Linux. Every step below includes the exact command used, the result I observed, a screenshot as evidence, and a short note on why the finding matters from an attacker's point of view.
3. Tools Used
The table below lists each tool used in this report and its purpose.
Tool
Purpose
Kali Linux
Operating system used for all reconnaissance and scanning activities
whois
Find domain registration details (registrar, dates, name servers)
dig
Query DNS for the domain's A record and IP address
theHarvester
Collect public emails, hosts, IPs and names linked to the domain (OSINT)
whatweb
Fingerprint web technologies (server, CMS, plugins, IP)
dnsenum
Enumerate DNS records, test zone transfers and brute-force subdomains
Zenmap (Nmap GUI)
Run an Intense scan to find open ports, services, versions and OS guesses


4. Activities Performed
4.1 Footprinting & Reconnaissance
I performed reconnaissance against the networkwalks.com domain using five Kali Linux tools: whois, dig, theHarvester, WhatWeb and dnsenum. Each tool was used to collect a different type of information about the target.
WHOIS (whois networkwalks.com): The domain is registered through GoDaddy.com, LLC. It was created on 2019-11-06 and expires on 2027-11-06. The registrant details are hidden behind a privacy service (Domains By Proxy, LLC). The name servers are NS6135.HOSTGATOR.COM, NS6136.HOSTGATOR.COM, NS29.DOMAINCONTROL.COM and NS30.DOMAINCONTROL.COM. DNSSEC is unsigned.
dig (dig networkwalks.com): The query returned status NOERROR with one A record, resolving the domain to 192.232.216.135.
theHarvester (theHarvester -d networkwalks.com): The tool ran without finding anything. It reported no IPs, no emails, no people and no hosts, so no extra public information was exposed through its sources.
WhatWeb (whatweb networkwalks.com): The first request over HTTP was reset by the server, then the HTTPS request returned 200 OK. WhatWeb identified Apache as the web server, WordPress as the CMS (version 7.1.4 shown in the output) with WordPress Download Manager 3.3.70, jQuery 3.7.1, Google Tag Manager, the page title "Networkwalks Academy", the contact email info@networkwalks.com and the IP 192.232.216.135.
dnsenum (dnsenum networkwalks.com): The tool listed the host address, both HostGator name servers and the mail host mail.networkwalks.com (192.232.216.135). Zone transfer (AXFR) was attempted against ns6135.hostgator.com and ns6136.hostgator.com and was refused by both. Brute forcing found the subdomain autodiscover.networkwalks.com (192.232.216.135). The class C network range was identified as 192.232.216.0/24, and the reverse lookup on 256 addresses returned 0 results.
4.2 Network Scanning with Zenmap
For the second activity, I used Zenmap (Nmap 7.95) against networkwalks.com on 10 November 2026 with the Intense scan profile. The command was:
nmap -T4 -A -v networkwalks.com
The host was up (latency about 0.044 s) and resolved to 192.232.216.135, whose reverse DNS name is 192-232-216-135.unifiedlayer.com. Nmap scanned 1000 TCP ports and found 15 services. The main open ports and services were:
Port
Service
Version / detail
21/tcp
FTP
Pure-FTPd; anonymous FTP login allowed (code 230)
22/tcp
SSH
OpenSSH 9.9 (protocol 2.0)
25/tcp, 26/tcp, 587/tcp
SMTP
Exim smtpd 4.100.1
53/tcp
DNS
generic DNS response (NOTIMP)
80/tcp
HTTP
Apache httpd
110/tcp, 995/tcp
POP3 / POP3S
Dovecot pop3d
143/tcp, 993/tcp
IMAP / IMAPS
Dovecot imapd
443/tcp
HTTPS
web server (TLS)
2222/tcp
SSH
OpenSSH 9.9 (protocol 2.0)
3306/tcp
MySQL
MySQL 5.7.44-48

The scan also showed TLS certificates issued by RapidSSL, DigiCert and Let's Encrypt, and 985 closed ports. The OS detection did not find an exact match, and its guesses (such as a VoIP adapter or a virtual machine) are unreliable. The target is 1 network hop away. The scan completed in 167.33 seconds.
5. Risk Analysis / Impact
Based on the information collected during the footprinting and scanning activities, I identified the following potential risks.
#
Risk / Finding
Evidence / Observation
Potential Impact
Risk Level
1
Anonymous FTP login allowed
Nmap ftp-anon script reported anonymous login allowed (code 230) on port 21 (Pure-FTPd)
Unauthenticated users may be able to list or access files, which needs to be confirmed
High
2
MySQL exposed to the internet
Port 3306 open, MySQL 5.7.44-48 detected
Database service reachable from outside; the 5.7 series is end of life, so it needs review
Medium
3
Many remote access and mail services exposed
SSH (22, 2222), SMTP (25, 26, 587), POP3/IMAP (110, 143, 993, 995) open
Larger attack surface that may be targeted by brute force or enumeration
Medium
4
Web technology information exposed
WhatWeb identified WordPress and WordPress Download Manager with version numbers
Attackers may use version information to look for known issues in the CMS and plugin
Medium
5
DNS infrastructure information exposed
dnsenum found name servers, mail host, autodiscover subdomain and the 192.232.216.0/24 range
Helps build a broader infrastructure profile
Low
6
Server IP address identifiable
dig and dnsenum resolved the domain to 192.232.216.135
Reveals the network location of the web service
Low
7
Domain registration details visible
WHOIS shows registrar, dates and name servers; owner details are privacy protected
Limited exposure; privacy protection reduces the risk
Low
8
DNSSEC not enabled
WHOIS shows DNSSEC: unsigned
DNS responses are not cryptographically signed
Low

Zone transfer was refused by both name servers, which is the correct and secure behaviour.
Risk level key: High, Medium, Low.
The risks above are observations from the footprinting and scanning exercises, not confirmed vulnerabilities. The practical exercises primarily involved information gathering and port and service discovery. No exploitation or vulnerability validation was performed as part of these two modules. Therefore, the presence of a software version, open port or DNS record does not by itself mean that the system is vulnerable. Further authorized security testing would be required to confirm any actual vulnerability.
6. Recommendations
Based on the observations from these activities, I recommend the following security improvements:
Disable anonymous FTP. If FTP is needed, require authenticated users, or replace it with SFTP.
Restrict the MySQL port. Limit port 3306 to trusted IP addresses or localhost, and plan an upgrade from the end-of-life 5.7 series.
Reduce exposed services. Close ports that are not needed, and restrict SSH and mail services with firewall rules, key-based login and rate limiting.
Keep software updated. WordPress, plugins and server software should be updated regularly and checked against current security advisories.
Review exposed technology information. Limit the version details that the website shows about its CMS and plugins.
Review DNS records regularly. Make sure only required records and subdomains are public, and consider enabling DNSSEC.
Keep zone transfers restricted. Both name servers correctly refused AXFR, and this should stay in place.
Monitor and log scanning activity. Unusual scans and connection attempts should be detected and reviewed.
Perform security testing with authorization. Reconnaissance and scanning should only be performed against systems and networks where appropriate authorization has been provided.

7. Conclusion
During Week 2 of my Cybersecurity & Ethical Hacking internship, I completed practical activities covering footprinting, reconnaissance and network scanning.
In the footprinting activity, I used five Kali Linux tools to collect information about the target domain. I learned how whois can provide domain registration information, dig can resolve a domain to its IP address, theHarvester can search for public emails and hosts, WhatWeb can identify web technologies, and dnsenum can enumerate DNS records and test for zone transfers.
In the network scanning activity, I used Zenmap with the Intense scan profile to discover open ports, running services, versions and operating system guesses on the target. The scan showed a wide range of exposed services, including FTP with anonymous login allowed, SSH, mail services and MySQL.
The exercises showed me that information gathering is an important part of cybersecurity. Even before attempting to exploit a system, a security professional can learn a significant amount about an environment by carefully analyzing publicly available information and network responses.
I also learned that technical findings should be documented clearly. A good cybersecurity report should explain what was performed, what was discovered, what the observation means, what risk it may create, and what can be done to reduce that risk.
Finally, I learned that reconnaissance and scanning must always be performed within an authorized scope. These activities were completed as part of the assigned educational cybersecurity lab.

