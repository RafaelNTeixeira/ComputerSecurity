# Trabalho realizado nas Semanas #2 e #3

## Identificação

- CVE-2023-27350 - Authentication Bypass (the attacker bypasses the authentication process entirely, being able to escalate privileges, move to other pages, steal or alter data, or download malicious firmware)

- This vulnerability remote attackers to bypass authentication and execute arbitrary code on affected installations of PaperCut

- The specific flaw exists within the SetupCompleted class, resulting from improper access control

- Affected PaperCut MF or NG systems:
    - version 8.0.0 to 19.2.7 (inclusive)
    - version 20.0.0 to 20.1.6 (inclusive)
    - version 21.0.0 to 21.2.10 (inclusive)
    - version 22.0.0 to 22.0.8 (inclusive)


## Catalogação

- This orchestrated attack was organized by the exploiters Mohin Paramasivam and MaanVader, representatives of the Bl00dy Ransomware Gang. 

- This exploit garnered an exceptionally high CVSS score of 9.8 out of 10, signifying its extreme criticality.

- On 10/01/2023, a vulnerability in PaperCut was identified and reported by a third-party company known as Trend Micro. 

- Despite the prior warnings, on 19/04/2023, compelling evidence surfaced indicating that unpatched servers were under active exploitation in the real world, where it all began.

## Exploit

- In this type of CVE we found two different exploits (Authentication Bypass and Remote Code Execution), but we are only going to talk about the RCE exploit.

- There are two principal payloads defined on the code (group_payload and final_payload):
    - group_payload is used to manipulate the configuration of user groups 
    - final_payload is used to execute a remote shell command on the targeted system giving the ability to explore vulnerabilities and gain remote control over PaperCut NG/MG 22.0.4.

- The exploit automatizes the following steps:
    1. Initializes a HTTP session on the targeted server 
    2. Visits the first URL to configure the session 
    3. Visits the second URL to access the control panel 
    4. Sends a POST solicitation to another URL with the payload “final_payload” to gain RCE.


## Ataques

- Bl00dy Ransomware Gang gained access to victim networks across the Education Facilities Subsector and exposed PaperCut servers vulnerable to CVE-2023-27350 to the internet.

- Furthermore, they stole data and encrypted victim systems, leaving ransom notes demanding payment in exchange for the decryption of encrypted files.

- Additionally, the Bl00dy actors are said to have used TOR and other proxies from within victim networks for external communications in an attempt to mask malicious traffic and avoid detection
