
# 04 - RDP Compromise and MITRE ATT&CK Mapping

## Objective

Develop a realistic cyberattack scenario involving RDP and map the attack flow using the updated MITRE ATT&CK Enterprise Matrix. Steal confidential corporate data and delay detection by erasing evidence of the intrusion.

Brute force > 
Intern RDP login > 
Password in PDF > 
Manager RDP login > 
PowerShell exfil > 
Log clearing

Apply threat intelligence, Wazuh log analysis, and custom detection engineering to investigate and detect selected activities.

## Incident Story

An external attacker found an exposed Remote Desktop Protocol (RDP) service on a corporate Windows endpoint. They brute-forced the password of a low-privilege local account, Intern, and logged in.

On the Intern desktop, the attacker found an unencrypted file, Employee_Onboarding.pdf. It held the default password for a newly created IT Manager account (Kevin) that has local administrator rights.

The attacker opened a new RDP session as the Manager account and found unencrypted sensitive data, Trade Secret.docx. They opened PowerShell and ran a custom TCP socket script that streamed the file to a Netcat listener on their own machine. This avoids browser monitoring.

Before disconnecting, the attacker cleared the Windows Security and Sysmon event logs.

## Scope

### MITRE ATT&CK Mapping

I mapped the attack to the updated MITRE ATT&CK Enterprise Matrix using ATT&CK Navigator. The layer has nine techniques, in attack order. A technique and its sub-technique count as one step.

**Navigator layer:** ![SVG capture](../screenshots/screencapture_file.jpeg)

This is a mapping of a hypothetical scenario. Not every technique has log evidence in my lab.

### Techniques in attack sequence

| # | Tactic | ID | Technique | My attack |
| --- | --- | --- | --- | --- |
| 1 | Initial Access | T1133 | External Remote Services | RDP exposed to the attacker |
| 2 | Credential Access | T1110.001 | Brute Force: Password Guessing | Repeated guesses against the Intern account |
| 3 | Credential Access | T1552.001 | Unsecured Credentials: Credentials In Files | Default password read from Employee_Onboarding.pdf |
| 4 | Privilege Escalation | T1078.003 | Valid Accounts: Local Accounts | Login as the Manager account (local admin) |
| 5 | Collection | T1005 | Data from Local System | Trade Secret.docx found on the Manager desktop |
| 6 | Execution | T1059.001 | Command and Scripting Interpreter: PowerShell | TCP socket script run in PowerShell |
| 7 | Exfiltration | T1048.003 | Exfiltration Over Alternative Protocol: Exfiltration Over Unencrypted Non-C2 Protocol | Raw TCP stream to Netcat on port 4444 |
| 8 | Defense Impairment | T1685.005 | Disable or Modify Tools: Clear Windows Event Logs | wevtutil used on the Security and Sysmon logs |

## Wazuh Investigation

For each stage of the attack I picked the Windows or Sysmon event that should exist, searched for it in Wazuh, and recorded the fields.

### Lab Environment

| Component | Role | Detail |
| --- | --- | --- |
| Windows endpoint | Target | Hostname securedot, Wazuh agent windows_agent, IP 192.168.253.1, Sysmon installed |
| Kali Linux | Attacker | IP 192.168.253.128, workstation name kali, Netcat listener on port 4444 |
| Ubuntu Server | Wazuh manager | Hostname Wazuhserver |

All testing ran on an isolated lab network.




