# Threat-Modelling
Lab 3 – Threat Modeling of a Small WLAN
Nov 24, 2026 · @Shaq
1. Objective and scenario
For this lab I looked at a small office Wi-Fi network the way an attacker would. The idea was to work out what is worth stealing or breaking, how someone could get at it, and what would stop them.
Objective. List the assets on a small WLAN, work out the threats and weak spots, and suggest controls. I used a paper diagram first and then OWASP Threat Dragon.
Scenario (assumed). A small design office with 10 staff, one owner and occasional visitors. All traffic passes through one combined router/access point (AP) supplied by the ISP.
• One Wi-Fi network, WPA2-Personal, shared password written on the office whiteboard
• 10 laptops, 6 smartphones, 1 network printer, 1 NAS file server holding client projects
• Router admin page still on default credentials; firmware last updated 2 years ago
• Guests use the same Wi-Fi as staff; no VLANs, no guest network
• Cloud services reached over the internet: email, file sync, accounting
Method. I started with the assets, drew a diagram showing where the trust boundary sits, went through each part with STRIDE, then scored the risks and matched controls to them.
Scope. I covered the Wi-Fi network, the devices on it, the router/AP and the data that passes through them. I left out the ISP's network and the security of the cloud providers themselves.
2. Asset identification
The assets that matter most are the files on the NAS, the Wi-Fi and router passwords, and the router itself. Each one is rated High, Medium or Low for confidentiality (C), integrity (I) and availability (A).
ID
Asset
Type
C
I
A
Why it matters
A1
Client project files on NAS
Data
High
High
High
Business revenue; client contracts require confidentiality
A2
Wi-Fi password (PSK)
Credential
High
Medium
Medium
Opens the whole network to anyone who has it
A3
Router/AP admin account
Credential
High
High
Medium
Full control of the network and DNS
A4
Router/AP firmware and config
System
Medium
High
High
Compromise gives a persistent foothold
A5
Staff laptops
Device
High
High
High
Hold email, saved passwords, local copies of files
A6
Staff smartphones
Device
Medium
Medium
Low
Email, 2FA apps, work chat
A7
Network printer
Device
Low
Low
Medium
Stores scanned documents; common pivot point
A8
Cloud logins (email, accounting)
Credential
High
High
Medium
Financial data and impersonation risk
A9
Wireless traffic in transit
Data
High
High
Medium
Contains logins, files, email
A10
Internet connection
Service
Low
Low
High
Office stops working without it
A11
Staff and guest identities
People
Medium
Medium
Low
Targets for phishing and social engineering
3. Architecture and data flow diagram
The office has one trust boundary. Everything inside it (staff devices, printer, NAS and router) can talk to everything else, which is part of the problem.
Anyone who gets onto the Wi-Fi ends up inside that boundary and can reach the NAS directly. The router is the only link between the office and the internet.
Build it in OWASP Threat Dragon
1.  Create a new threat model titled Lab 3 Small WLAN and add a STRIDE diagram.
2.  Add two external entities (Attacker or guest, Internet), three processes (Router/AP, Staff devices, Printer) and one data store (NAS server).
3.  Draw the data flows: Attacker to Router, Router to Internet, and Router to each of Staff devices, Printer and NAS. Label each flow with its protocol, for example Wi-Fi WPA2 or SMB.
4.  Draw one trust boundary around Router, Staff devices, Printer and NAS.
5.  Select each element and add the threats from section 5: STRIDE type, description, severity and mitigation. Link each mitigation to a control from section 7.
6.  Export the diagram as PDF or PNG and the model as JSON for submission.
Paper-based version. Draw the same elements on A4: circles for processes, two parallel lines for the data store, rectangles for external entities, a dashed line for the trust boundary. Write the protocol and encryption on each arrow, then add T-numbers next to the elements they affect.
4. Attacker view
An attacker will usually take the easiest route to the most valuable thing. In this office that means the shared Wi-Fi password or the unpatched router, not some clever exploit.
Attacker
Motivation
Skill
Likely entry point
Curious neighbour or passer-by
Free internet, curiosity
Low
Guessed or shared Wi-Fi password
Malicious visitor or guest
Data theft, mischief
Low–Medium
Joins the same Wi-Fi, scans the LAN, reaches the NAS
Disgruntled ex-employee
Revenge, data theft
Medium
Still knows the PSK; password never changed
Opportunistic criminal in the car park
Ransom, resale of data
Medium
Evil twin AP, handshake capture, router default login
Targeted attacker
Client data, fraud
High
Phishing a laptop, then moving across the flat network
Typical attack paths
1.  Capture the WPA2 handshake from outside the building, crack a weak password offline, join the network, reach the NAS.
2.  Set up a fake AP with the office SSID near the entrance; staff phones connect; the attacker reads or alters their traffic.
3.  Open the router admin page with default credentials, change DNS to a malicious server, redirect staff to fake login pages.
4.  Sit as a guest on the flat network, scan for open SMB shares and the printer, copy files.
5.  Flood the AP with deauthentication frames to knock devices offline during working hours.
5. Threats and vulnerabilities
I found twelve threats by going through each asset and data flow with STRIDE (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege).
ID
STRIDE
Threat
Vulnerability that enables it
Asset
T1
Spoofing
Evil twin AP impersonates the office SSID
Clients auto-join by SSID name; no server authentication in WPA2-Personal
A6, A5, A9
T2
Information disclosure
Offline cracking of captured WPA2 handshake
Weak, shared PSK that is rarely changed
A2, A9
T3
Information disclosure
Eavesdropping on unencrypted traffic (HTTP, old protocols)
Some services use no TLS; all users share the same key material
A9
T4
Elevation of privilege
Takeover of router admin account
Default credentials; admin page reachable from Wi-Fi
A3, A4
T5
Tampering
DNS hijack or malicious router config change
Router compromised; no config change alerts
A4, A9
T6
Elevation of privilege
Exploit of known router firmware flaw
Firmware 2 years old; no update process
A4
T7
Information disclosure
Guest or intruder reads NAS files
Flat network, guests and staff share one segment; weak NAS share permissions
A1
T8
Denial of service
Deauthentication flood or radio jamming
Management frames unprotected on WPA2 without 802.11w
A10, A5
T9
Tampering
Ransomware spreads from one laptop to the NAS
No segmentation; no offline backup
A1, A5
T10
Spoofing
Rogue device plugged in or joined; staff impersonated
No device inventory or MAC/port control
A5, A11
T11
Repudiation
Malicious action cannot be traced to a person
No logging; one shared password for everyone
A2, A11
T12
Information disclosure
Scanned documents read from the printer
Printer on default settings with open web interface
A7
Main weaknesses: a weak shared Wi-Fi password, default admin login, old firmware, no network segmentation, no logging, no backups and unprotected management frames.
6. Risk rating
Five threats came out as Critical or High (T4, T2, T7, T9 and T6), so those get fixed first. Risk is likelihood times impact, each scored from 1 to 5. A score of 15 or more is Critical/High, 8 to 14 is Medium and 7 or less is Low.
Threat
Likelihood
Impact
Score
Rating
T4 Router takeover via default login
4
5
20
Critical
T2 PSK cracked offline
4
4
16
High
T7 Intruder reads NAS files
4
4
16
High
T9 Ransomware reaches NAS
3
5
15
High
T6 Router firmware exploit
3
5
15
High
T1 Evil twin AP
3
4
12
Medium
T5 DNS hijack
2
5
10
Medium
T3 Eavesdropping on unencrypted traffic
3
3
9
Medium
T8 Deauthentication flood
3
3
9
Medium
T10 Rogue device or impersonation
2
4
8
Medium
T11 No accountability (no logs)
3
2
6
Low
T12 Printer data exposure
2
2
4
Low
These scores are my own judgment for the scenario I assumed, so they could change if the lab brief gives different numbers.
7. Proposed controls
The quickest wins are changing the router password, updating the firmware, keeping guests apart from staff and moving to WPA3 or WPA2-Enterprise. The controls below are in priority order, and each one lists the threats it helps with.
Priority
Control
Type
Threats reduced
1
Change router admin password; disable remote and Wi-Fi-side admin access
Preventive
T4, T5
2
Update router firmware now; enable automatic updates or a monthly check
Preventive
T6, T4
3
Move to WPA3-Personal (or WPA2-Enterprise with 802.1X/RADIUS if supported); use a random passphrase of 16+ characters
Preventive
T1, T2, T3, T10, T11
4
Create a separate guest SSID with client isolation and internet-only access
Preventive
T7, T9
5
Segment staff, NAS and printer with VLANs; firewall rules allow only needed ports
Preventive
T7, T9, T12
6
Enable 802.11w (Protected Management Frames)
Preventive
T8, T1
7
3-2-1 backups with one offline or immutable copy; test restores quarterly
Corrective
T9
8
Harden NAS: per-user accounts, least-privilege shares, disable SMB1, enable 2FA
Preventive
T7, T9, T11
9
Enforce HTTPS/TLS and a VPN on untrusted networks; turn off legacy protocols
Preventive
T3, T1
10
Turn on router and NAS logging; alert on new devices and config changes
Detective
T5, T10, T11
11
Run a wireless scan for rogue APs every quarter
Detective
T1, T10
12
Change printer defaults, disable unused services, set an admin password
Preventive
T12
13
Staff awareness: phishing, spotting fake Wi-Fi, never sharing the password
Preventive
T1, T9, T10
14
Incident plan: who to call, how to isolate a device, how to rotate the Wi-Fi key
Corrective
All
Residual risk. If controls 1 to 8 are done, T4 should fall from 20 to about 4, T2 from 16 to about 6 and T7 from 16 to about 4. Jamming (T8) can't really be stopped, so I would accept it because the impact is low.
Evidence from Kali Linux (host: Shadrack)
Each figure below should be a screenshot from my Kali Linux machine (hostname Shadrack), taken only on my own lab network or one I have permission to test.
Fig.
Purpose
Command to run on Shadrack
What the screenshot should show
1
Set and confirm hostname
sudo hostnamectl set-hostname Shadrack then hostnamectl and ip a
Hostname Shadrack, your IP and subnet (maps to A5)
2
Find your wireless interface
iwconfig or iw dev
Interface name such as wlan0
3
Passive survey of nearby APs
sudo airmon-ng start wlan0 then sudo airodump-ng wlan0mon
Your SSID, its encryption (WPA2/WPA3), channel, and any look-alike SSIDs (T1, T2)
4
Discover devices on your lab LAN
nmap -sn 192.168.1.0/24 (use your subnet)
Router, NAS, printer, laptops listed (A3 to A7)
5
Check open services on router, NAS and printer
nmap -sV 192.168.1.1 192.168.1.20 (your addresses)
Open ports such as 80, 443, 445, showing the exposed admin page and SMB share (T4, T7, T12)
6
Confirm no segmentation
ping a staff device and the NAS from the guest Wi-Fi
Replies prove guests can reach internal hosts (T7)
7
Verify a control after fixing
Repeat figures 5 and 6 after changing router password and enabling guest isolation
Fewer open ports, ping fails from guest (section 7, controls 1 to 5)
Figure 1 placeholder. Insert screenshot of hostnamectl output showing Shadrack.
Figure 3 placeholder. Insert screenshot of airodump-ng output.
Figure 5 placeholder. Insert screenshot of nmap service scan.
Figure 7 placeholder. Insert before and after screenshots.
Note. The Kali prompt reads kali@Shadrack, which shows the screenshots are mine. I only test my own network, never other people's networks or handshakes.
8. Conclusion
Doing the threat model showed me that this network is weak mostly because of basic things rather than advanced attacks: a shared password, a default router login, old firmware and a single flat network.
How attackers think. They go after the weakest link, not the strongest lock. They also chain small weaknesses together, for example guest access, then the flat network, then the NAS, and they like routes that need no special skill.
What the model gave me. Twelve threats ranked by risk, 14 controls linked to them and a clear order for fixing things: router password and firmware first, then segmentation, stronger Wi-Fi authentication and backups.
Limits. The scenario is assumed and the scores are my judgment. A real assessment would check them with a wireless survey and a review of the router and NAS settings.
