
# Trabalho realizado nas Semanas #2 e #3

## Identificação

- item1
- item2
- item3
- item4

## Catalogação

- item1
- item2
- item3
- item4

## Exploit

- In this type of CVE we found two different exploits (Authentication Bypass and Remote Code Execution), but we are only going to talk about the RCE exploit.

- There are two principal payloads defined on the code (group_payload and final_payload). The first one is used to manipulate the configuration of user groups and the last one to execute a remote shell command on the targeted system giving the ability to explore vulnerabilities and to gain remote control over PaperCut NG/MG 22.0.4.

The exploit automatizes the following steps:
- Initializes a HTTP session on the targeted server -> visits the first URL to configure the session -> visits the second URL to access the control panel -> sends a POST solicitation to another URL with the payload “final_payload” to gain RCE.


## Ataques

- item1
- item2
- item3
- item4
