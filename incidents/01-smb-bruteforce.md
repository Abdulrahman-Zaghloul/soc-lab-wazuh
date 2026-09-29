# Incident Report: SMB Brute-Force Attack — DESKTOP-QS8K66A

## Executive Summary
A brute-force attack against SMB authentication on host DESKTOP-QS8K66A
succeeded in compromising the local account `vboxuser`, using a weak,
easily-guessed password. The attack was detected by Wazuh's built-in
correlation rule for repeated logon failures.

## Environment
- **Target:** DESKTOP-QS8K66A (192.168.100.30) — Windows 10, Build 19041
- **Attacker source:** Kali Linux (192.168.100.20)
- **SIEM:** Wazuh (manager at 192.168.100.10)
- **Attack tool:** netexec (SMB module)

## Timeline

| Time | Event |
|---|---|
| ~10:56:56 | Attack begins — repeated SMB authentication attempts from 192.168.100.20 |
| ~10:56:56 | Wazuh logs first "Logon Failure - Unknown user or bad password" (rule 60122) |
| ~10:56:56 | Six total failed attempts logged in rapid succession |
| ~10:56:56 | Wazuh correlation rule 60204 "Multiple Windows Logon Failures" fires (level 10) |
| ~10:56:56 | Successful logon (Event ID 4624) confirmed for account vboxuser, logon type 3 |

## Technical Details
- **Protocol/Port:** SMB, TCP/445
- **Target account:** vboxuser
- **Compromised password:** kali (weak/dictionary password)
- **Logon type:** 3 (network logon — consistent with SMB)
- **SMB signing:** Disabled on target (confirmed via netexec fingerprinting) —
  an independent weakness beyond the password issue

## Detection
Wazuh's default ruleset correlated six individual low-severity
"Logon Failure" events (rule 60122, level 5) into a single higher-severity
alert (rule 60204, level 10) once a threshold of repeated failures was
reached within a short window. This is the built-in equivalent of
brute-force detection logic.

The alert was recorded and visible in the Wazuh dashboard, but no real-time
notification (email, chat, ticket) was configured, so an analyst would only
have seen it by actively reviewing the console.

## MITRE ATT&CK Mapping

| Tactic | Technique | Evidence |
|---|---|---|
| Discovery | T1046 Network Service Discovery | Nmap scan confirming SMB on TCP/445 |
| Credential Access | T1110.001 Brute Force: Password Guessing | Repeated failed logons (rule 60122) from 192.168.100.20 |
| Initial Access / Lateral Movement | T1078.003 Valid Accounts: Local Accounts; T1021.002 Remote Services: SMB/Windows Admin Shares | Successful type-3 logon (Event ID 4624) as `vboxuser` |

## Root Cause
1. Weak password policy — `kali` is a trivially guessable password
2. No observed account lockout policy — the account was not locked
   despite 6+ rapid failed attempts
3. SMB signing disabled, increasing exposure to relay-style attacks
   (separate from this specific incident, but discovered during
   investigation)

## Recommendations
1. Enforce a strong password policy (minimum length, complexity)
2. Enable account lockout after a small number of failed attempts
   (e.g., 5 within 5 minutes)
3. Enable SMB signing to prevent relay attacks
4. Consider disabling SMBv1 entirely if not already
5. Tune Wazuh alerting thresholds/notifications so rule 60204-level
   alerts trigger real-time analyst notification, not just logging

## Evidence

![Netexec successful credential crack](<evidence/Netexec success comaand.png>)

![Wazuh search showing failed logon cluster](<evidence/Wazuh failed logon search.png>)

![Wazuh inspection of a single failed logon event](<evidence/Wazuh failed logon inspection.png>)

![Nmap scan confirming SMB port 445 open](<evidence/nmap smb scan.png>)

![Wazuh search showing successful logon event](<evidence/Wazuh successful logon search.png>)

![Wazuh inspection of the successful logon event](<evidence/Wazuh successful logon inspection.png>)