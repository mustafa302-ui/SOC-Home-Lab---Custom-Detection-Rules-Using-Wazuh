# SOC-Home-Lab---Custom-Detection-Rules-Using-Wazuh
Custom Wazuh rules for detecting: brute force, nmap scans, and privilege escalation, with screenshots.

Project Overview

Title: Wazuh Detection & Correlation Lab

Description:
This project simulates real-world attacker behavior (scans, brute force, privilege escalation) against a Linux VM with the Wazuh agent installed. Custom correlation rules were written to detect suspicious patterns missed by defaults, including multi-stage escalation detection.

Key Skills Demonstrated:
- SIEM engineering with Wazuh
- Log analysis & field mapping (data.srcip, location)
- Custom correlation rule design (<if_matched_sid>, <same_source_ip/>, <same_location/>)
- MITRE ATT&CK mapping
- Hands-on attack simulation (Nmap, Hydra, sudo escalation loops)

Architecture: 
- Host: Ubuntu VM (192.168.64.136) with Wazuh agent
- Attacker: Kali Linux VM for scans/brute force
- Manager: Wazuh server (receives logs, applies custom rules)
- Log sources: /var/log/auth.log, Apache2 access/error logs, Wazuh agent internal queue
- (Insert a simple diagram here: Kali → Ubuntu (agent) → Wazuh Manager → Kibana/alerts) 

CURSTOM RULES IMPLEMENTED

SSH Brute Force (100100):
Description: ≥6 failed SSH logins from the same IP in 120s.
Why: Detects external brute force attempts.

MITRE: T1110.
Test Command: 
hydra -l testuser -P /usr/share/wordlists/rockyou.txt ssh://192.168.64.136 
sudo hydra -l root -P badpw.txt -t 4 -V ssh://192.168.64.136
------------------------------------------------------------------------------------

Apache Web Scanning (100200):
Description: ≥5 HTTP 400/501 errors in 120s from same IP.
Why: Detects recon attempts against Apache.
MITRE: T1595 (Active Scanning).

Test Command:
sudo nmap --script http-enum -p80 192.168.64.136 
------------------------------------------------------------------------------------

Queue Overflow / Event Flood (100201)

Description: ≥5 queue overflow warnings (SID 203) in 100s.
Why: Indicates possible DoS/evasion by noise.

MITRE: T1499.
Test Command:
nmap -sS -T4 -p- 192.168.64.136 
(Generates thousands of events rapidly, triggering 203 alerts.)  
<img width="623" height="276" alt="Screenshot 2025-09-08 005659" src="https://github.com/user-attachments/assets/c9824d23-50a7-495c-9da7-d4bdcae4a175" />

------------------------------------------------------------------------------------ 

Multi-Stage Privilege Escalation (100311, 100312, 100301)

Child A (100311): ≥4 login failures in 5m (SID 5503).
Child B (100312): ≥2 sudo-fail bursts (SID 5404) in 5m.
Parent (100301): Fires if both child patterns occur in ≤5m.

Why: Escalation risk is high if brute force + sudo failures happen together.
MITRE: T1110 + T1548.

Test Command:
for i in {1..6}; do sudo -k; sudo ls /root; done 
<img width="1836" height="672" alt="Screenshot 2025-09-08 001100" src="https://github.com/user-attachments/assets/b644e9c0-e5e7-4f9f-9302-9a96ffd314a0" />

------------------------------------------------------------------------------------ 

Lessons Learned

- Always confirm where fields live (srcip vs. data.srcip).
- Don’t mix raw SIDs with already-aggregated rules unless using multi-stage correlation.
- <same_location/> is safer than <same_field> in auth cases.
- Build child → parent rules to highlight true positive escalation scenarios.
------------------------------------------------------------------------------------
