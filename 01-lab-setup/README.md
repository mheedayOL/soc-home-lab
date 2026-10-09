# 01: Lab Setup

## Objective
To build, baseline and investigate a small IT environment

## Environment
- Host machine: [HP Elitebook Folio 1040 G3, 8GB RAM, core i5]
- VMs: Ubuntu Server (version 24.04.5), Windows 10, Kali Linux
- Network setup: [NAT]

## Steps
1. First, I downloaded the Ubuntu server [version 24.04.5.1] at  https://ubuntu.com/download/server
2. Next, I created a new virtual machine in my VMware and installed the downloaded Ubuntu server.

3. | Hostname | OS | IP | MAC | Role | Network Adapter Type | Default gateway | DNS |
   | --- | --- | --- | --- | --- | --- | --- | --- |
   | Ubuntu 64-bit | Ubuntu 24.04.5 LTS | 192.168.253.129/24 | 00:0c:29:19:e0:85 | SIEM | NAT | 192.168.253.2 | 192.168.253.2 |

4. Then https://www.microsoft.com/en-us/software-download/windows10 to create the installation media to download the windows 10 iso image
5. Next, I created another virtual machine and installed the windows 10 iso on it.

      ![Windows](../screenshots/windows.png)
   
6. | Hostname | OS | IP | MAC | Role | Network Adapter Type | Default gateway | DNS |
   | --- | --- | --- | --- | --- | --- | --- | --- |
   | DESKTOP-M52JJJH | Windows 10 pro | 192.168.253.131 | 00:0c:29:19:e0:85 | Endpoint | NAT | 192.168.253.2 | 192.168.253.2 |

   Running  Processes on my WIndows machine ![Running Processes](../screenshots/running-processes.png)

   | Name/PID | What it Does | Why It Matters | Expected | Suspicious If |
   | --- | --- | ---| ---| --- |
   | Explorer.exe / PID 948 | It is the windows shell | It runs as my user, so it inherits permissions | Yes, normally one instance | It runs from another foldeer or a second copy appears unexpectedly |
   | svchost.exe / PID 2412,4360 | Shared host that runs many windows services | there are many copies so malware can hide itself by naming itself svchost.exe | Yes, many instances | The path is outside System32, the parent isn't services.exe or making unusual network connections |
   | Powershell.exe / PID 5364 | command shell | Most SOC detections for it focus on how it was launched | Yes, because I opened it myself | it runs when no user is logged in |
   | conhost.exe / PID 3760 | Draws console window for command-line programs | every command-line session gets one | Yes, it belongs to my powershell window | it exists with no matching console program |
   | RuntimrBroker.exe / PID 396,1964, | manages permission checks between microsoft apps and the system | it sits between apps so its name is easy to fake | Yes, several instances | the path is outside System32, the spelling is subtly different or it keeps using high CPU. 

   Running Services ![Running services](../screenshots/running-services.png)

   Listening ports ![listening ports](../screenshots/listening-ports.png)

   Active Connections ![Active connections](../screenshots/active-connections.png)

   Firewall Status ![Firewall status](../screenshots/firewall-status.png)

   Log evidence ![System log](../screenshots/system-log.png)
   
7. Next, I created another virtual machine for Kali Linux
  
8. | Hostname | OS | IP | MAC | Role | Network Adapter Type | Default gateway | DNS |
   | --- | --- | --- | --- | --- | --- | --- | --- |
   | Kali | Kali GNU/Linux 2025.4 | 192.168.253.128/24 | 00:0c:29:15:a1:b7 |Security Testing | NAT | 192.168.253.2 | 192.168.253.2 |

9. Created at least two users and one group; added and removed a user from the group
![User created](../screenshots/User-created.png)

10. Created a test file/directory. Allow one user, restrict another, change ownership and modes, test access
![Test directory](../screenshots/Test-directory.png)

11. Who can access it, with what permissions, and why.

| Item | Mode | Owner/group | Who can do what | Why |
| --- | --- | --- | --- | --- |
| /srv/soc_data | 770 | root / soc_team | root and soc_team members can list, enter and create files. Others get nothing. | Owner and group get rwx, others get ---. A user needs x on a directory to enter it. |
| report.txt |	640 | alice / soc_team | alice can read and write. Group members can only read. Others get nothing. | Owner rw-, group r--, others ---. |
| bob (not in group) |  |  | Blocked at the directory | bob falls under "others", which has no permissions. |
| bob (in group) |  |  | Can read the file, cannot write | The group has read-only access on the file. |

12. normal logins ![Normal login](../screenshots/normal-login.png)
13. I ran a few controlled failed logins against my own lab. ![Failed logins](../screenshots/failed-login.png)
14. Then I checked the evidence in the logs ![here](../screenshots/log-evidence.png)

## Results
Successfully installed 3 VMs [Ubuntu Server, Windows 10 and Kali Linux]
Established a connection between all 3 VMs

![Kali to Windows](../screenshots/kali-to-windows.png)

![Windows](../screenshots/window-to-ubuntu.png)

![Ubuntu](../screenshots/ubuntu-to-kali.png)

Network Baseline (Wireshark)
I captured normal traffic(ping, DNS lookup, web/HTTPS, a TCP connection, SSH to ubuntu machine)

![packet capture](../screenshots/packet-capture.png) 

According to the image above, I found the following information for some selected activity

| Source IP | Destination IP | Protocol | Ports | Key Packet Information | activity that generated it | why it appears in Wireshark |
| --- | --- | --- | --- | --- | --- | --- |
| 192.168.253.128 | 192.168.253.129 | TCP | src 36696 dst 22 | Flags (ACK) Window size 256  | ssh into ubuntu machine | These packets appear because the two machines are opening the connection | 
| 176.97.192.150 | 192.168.253.128 | NTP | src 123 dst 46182 | Flags 0x24 Leap Indicator: no warning | Ubuntu server pinged Kali Machine | normal background traffic | 

Controlled Security Activity
I completed two controlled security activities against the Ubuntu Server from the Kali Linux VM.

Nmap Port Scan from kali vm against ubuntu vm 
To perform controlled reconnaissance and identify exposed ports/services on the Ubuntu server.

![Nmap scan results](../screenshots/nmap-scan.png)

Controlled Failed SSH Authentication
To generate controlled authentication-failure activity and observe the evidence produced by the SSH service.
Activity:Entered an incorrect password 2 times.
Evidence: Permission denied` on Kali ![Denied](../screenshots/denied.png) 

failed authentication entries in the Ubuntu SSH logs. ![SSH logs](../screenshots/ssh-logs.png)

# Before vs After
## Baseline

What I observed:
- `ssh.service` starts and listens on port 22
- Successful password logins

## Change (controlled activity)
- `nmap ubuntuIP` from Kali, a default port scan that lasted 2.45 seconds.
- Deliberate failed logins from my SSH log followed by a success.
  
## Comparison

| Field | Content |
|---|---|
| Observation | 5 `Failed password` events in total, 3 login sequences, all ending in success. Each sequence reuses one source port, so each is a single SSH connection. |
| Normal baseline | Clean `Accepted password` logins for known users, with no failures before them. |
| Controlled activity |  One default Nmap scan from Kali. Wrong passwords entered deliberately |
| Difference | New failure lines and a fail, fail, succeed pattern, with the same users and IPs. The source IPs and usernames stay the same as in the baseline. |
| Security relevance | Scanning is reconnaissance. Failed logins are used in detection rules. |

## Investigation

**Observed:**
- The scan found exactly two open services on the ubuntu vm
- Every failed attempt used a valid account. There are no `Invalid user` lines.
- No `root` attempts, and no other sources aside from my lab IPs.

**Concluded:**
- A success after failures is worth noting in a real SOC, but here it is low volume and from a known lab host for security testing.
They look abnormal against the baseline, but that doesn't make them malicious

