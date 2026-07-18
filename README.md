# APT Detection Lab

A SOC home lab that detects credential dumping — the same technique used by Russian military intelligence (APT28) against Ukrainian government organizations, documented by CERT-UA.

## Why I Built This

I am Ukrainian and it's a pity to see my country having to go through war. This war brought a lot of pain and destruction to people, cities, infrastractures and networks. I simulated this exact attack because Russian hackers attacked Ukraine using those same techniques I used in this project. I wanted to learn how they did it, what they used, and how to scan and protect against those same attacks in the future. 

This attack scenario was published on official Ukrainian national cybersecurity agency CERT-UA. Techniques, tools and strategy, all of it was published there, and I recreated this attack using Oracle VirtualBox Machine. I wanted to understand how this type of scenario works, what happens, and what is vulnaruble to the hackers.

## Lab Environment

| Component | Details |
|---|---|
| SIEM | Wazuh 4.14.5 |
| Domain Controller | Windows Server 2019 — Active Directory |
| Domain | MYDOMAIN.COM — 1,056 user accounts |
| Target Machine | Windows 10 Enterprise — CLIENT1.mydomain.com |
| Target IP | 172.16.0.100 |
| Wazuh Server IP | 172.16.0.101 |
| Detection Method | Windows Security Auditing — Process Creation (Event ID 4688) |
| Attack Tool | Mimikatz 2.2.0 x64 |

## Attack Scenario — T1003.001: OS Credential Dumping, LSASS Memory

LSASS is the Windows process that stores active user credentials in
memory: NTLM hashes, Kerberos tickets, and in some configurations
plaintext passwords for every account logged into the machine.

I used Mimikatz to access LSASS memory on the domain-joined workstation
CLIENT1 using the sekurlsa::logonpasswords module. The output returned
the username, NTLM hash, and plaintext password for the domain account
a-jbabushka in MYDOMAIN: credentials that could be used to authenticate
to any system that account has access to across the domain, without
knowing the actual password.

Windows Security Auditing captured the process creation as Event ID 4688,
logging the full path C:\Tools\mimikatz\x64\mimikatz.exe, the parent
process cmd.exe, and the account that launched it. The Wazuh agent
forwarded the event to the SIEM within seconds.

## Detection

**Mimikatz credential dump — domain hashes exposed:**

![Mimikatz Output](https://github.com/batihroman/apt-detection-lab/blob/45a085d621738e977de820467a47fc8d94df9b78/mimikatz-output.png.png)

**Wazuh SIEM alert — Event ID 4688 capturing Mimikatz process creation:**

![Wazuh Detection](https://github.com/batihroman/apt-detection-lab/blob/45a085d621738e977de820467a47fc8d94df9b78/wazuh-detection(2).png.png)

**Both agents forwarding logs to Wazuh:**

![Wazuh Agents](https://github.com/batihroman/apt-detection-lab/blob/45a085d621738e977de820467a47fc8d94df9b78/wazuh-agent-dashboard.png.png)

Full incident documentation:
[IR-001 — Credential Dumping Incident Report](https://github.com/batihroman/apt-detection-lab/blob/45a085d621738e977de820467a47fc8d94df9b78/incident-reports/IR-001-Credential-Dumping.md)

## Detection Details

| Field | Value |
|---|---|
| Windows Event ID | 4688 — Process Creation |
| Process | mimikatz.exe |
| Full Path | C:\Tools\mimikatz\x64\mimikatz.exe |
| Parent Process | C:\Windows\System32\cmd.exe |
| Account | a-jbabushka — MYDOMAIN |
| Machine | CLIENT1.mydomain.com |
| Wazuh Rule ID | 67027 |
| Wazuh Rule Level | 3 |
| Timestamp | 2026-06-18 02:04:07 UTC |

## MITRE ATT&CK Coverage

**T1003.001 — OS Credential Dumping: LSASS Memory**
Tactic: Credential Access

![MITRE ATT&CK Heatmap](https://github.com/batihroman/apt-detection-lab/blob/45a085d621738e977de820467a47fc8d94df9b78/mitre-heatmap.svg.svg)

## Threat Intelligence Reference

This lab is based on CERT-UA advisories documenting UAC-0028 — the CERT-UA tracking identifier for APT28, a Russian GRU military intelligence unit. APT28 has conducted credential harvesting campaigns against Ukrainian government ministries, energy infrastructure, military institutions, and diplomatic facilities throughout the Russia-Ukraine conflict.

Their standard operation: phishing for initial access on one machine, credential dumping to move laterally, domain controller compromise within one hour of the initial breach. Credential dumping via LSASS is the bridge between one compromised workstation and the entire network.

CERT-UA: https://cert.gov.ua
MITRE ATT&CK T1003.001: https://attack.mitre.org/techniques/T1003/001/
  custom Wazuh rule with a higher severity level specifically for
  Mimikatz

Write it honestly. Recruiters and admissions officers both respond
to genuine reflection more than polished statements.]
