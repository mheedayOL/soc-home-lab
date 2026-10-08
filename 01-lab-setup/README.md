# 01: Lab Setup

## Objective
To set up virtual machines that wil be useful for this home lab

## Environment
- Host machine: [HP Elitebook Folio 1040 G3, 8GB RAM, core i5]
- VMs: Ubuntu Server (version 24.04.5), Windows 10, Kali Linux
- Network setup: [NAT network]

## Steps
1. First, I downloaded the Ubuntu server [version 24.04.5.1] at  https://ubuntu.com/download/server
2. Next, I created a new virtual machine in my VMware and installed the downloaded Ubuntu server.

3. | Hostname | OS | IP | MAC | Role | Network Adapter Type | Default gateway | DNS |
   | --- | --- | --- | --- | --- | --- | --- | --- |
   | Ubuntu 64-bit | Ubuntu 24.04.5 LTS | 192.168.253.129/24 | 00:0c:29:19:e0:85 | SIEM | NAT | 192.168.253.2 | 192.168.253.2 |

4. Then https://www.microsoft.com/en-us/software-download/windows10 to create the installation media to download the windows 10 iso image
5. Next, I created another virtual machine and installed the windows 10 iso on it.
   
7. | Hostname | OS | IP | MAC | Role | Network Adapter Type | Default gateway | DNS |
   | --- | --- | --- | --- | --- | --- | --- | --- |
   | Windows | Ubuntu 24.04.5 LTS | 192.168.253.129 | 00:0c:29:19:e0:85 | Endpoint | NAT | 192.168.253.2 | 192.168.253.2 |

8. Next, I created another virtual machine for Kali Linux
  
9. | Hostname | OS | IP | MAC | Role | Network Adapter Type | Default gateway | DNS |
   | --- | --- | --- | --- | --- | --- | --- | --- |
   | Kali | Kali GNU/Linux 2025.4 | 192.168.253.128/24 | 00:0c:29:15:a1:b7 |Security Testing | NAT | 192.168.253.2 | 192.168.253.2 |

10. Created at least two users and one group; added and removed a user from the group
![User created](../screenshots/user-created.png)

11. Created a test file/directory. Allow one user, restrict another, change ownership and modes, test access
![Test directory](../screenshots/test-directory.png)

12. Who can access it, with what permissions, and why.

| Item | Mode | Owner/group | Who can do what | Why |
| --- | --- | --- | --- | --- |
| /srv/soc_data | 770 | root / soc_team | root and soc_team members can list, enter and create files. Others get nothing. | Owner and group get rwx, others get ---. A user needs x on a directory to enter it. |
| report.txt |	640 | alice / soc_team | alice can read and write. Group members can only read. Others get nothing. | Owner rw-, group r--, others ---. |
| bob (not in group) |  |  | Blocked at the directory | bob falls under "others", which has no permissions. |
| bob (in group) |  |  | Can read the file, cannot write | The group has read-only access on the file. |

13. normal logins ![Normal login](../screenshots/normal-login.png)
14. I ran a few controlled failed logins against my own lab. ![Failed logins](../screenshots/failed-login.png)
15. Then I checked the evidence in the logs ![here](../screenshots/log-evidence.png)

## Results
Successfully installed 3 VMs [Ubuntu Server, Windows 10 and Kali Linux]
Established a connection between all 3 VMs
Performed normal logins and failed logins and checked the evidence in the logs
