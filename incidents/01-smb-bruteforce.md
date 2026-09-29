\# Incident Report: SMB Brute-Force Attack — DESKTOP-QS8K66A



\## Executive Summary

A brute-force attack against SMB authentication on host DESKTOP-QS8K66A

succeeded in compromising the local account `vboxuser`, using a weak,

easily-guessed password. The attack was detected by Wazuh's built-in

correlation rule for repeated logon failures.



\## Environment

\- \*\*Target:\*\* DESKTOP-QS8K66A (192.168.100.30) — Windows 10, Build 19041

\- \*\*Attacker source:\*\* Kali Linux (192.168.100.20)

\- \*\*SIEM:\*\* Wazuh (manager at 192.168.100.10)

\- \*\*Attack tool:\*\* netexec (SMB module)



\## Timeline



| Time | Event |

|---|---|

| \~10:56:56 | Attack begins — repeated SMB authentication attempts from 192.168.100.20 |

| \~10:56:56 | Wazuh logs first "Logon Failure - Unknown user or bad password" (rule 60122) |

| \~10:56:56 | Six total failed attempts logged in rapid succession |

| \~10:56:56 | Wazuh correlation rule 60204 "Multiple Windows Logon Failures" fires (level 10) |

| \[PENDING] | Successful logon (Event ID 4624) confirmed for account vboxuser |



\## Technical Details

\- \*\*Protocol/Port:\*\* SMB, TCP/445

\- \*\*Target account:\*\* vboxuser

\- \*\*Compromised password:\*\* kali (weak/dictionary password)

\- \*\*Logon type:\*\* 3 (network logon — consistent with SMB)

\- \*\*SMB signing:\*\* Disabled on target (confirmed via netexec fingerprinting) —

&#x20; an independent weakness beyond the password issue



\## Detection

Wazuh's default ruleset correlated six individual low-severity

"Logon Failure" events (rule 60122, level 5) into a single higher-severity

alert (rule 60204, level 10) once a threshold of repeated failures was

reached within a short window. This is the built-in equivalent of

brute-force detection logic.



\## Root Cause

1\. Weak password policy — `kali` is a trivially guessable password

2\. No observed account lockout policy — the account was not locked

&#x20;  despite 6+ rapid failed attempts

3\. SMB signing disabled, increasing exposure to relay-style attacks

&#x20;  (separate from this specific incident, but discovered during

&#x20;  investigation)



\## Recommendations

1\. Enforce a strong password policy (minimum length, complexity)

2\. Enable account lockout after a small number of failed attempts

&#x20;  (e.g., 5 within 5 minutes)

3\. Enable SMB signing to prevent relay attacks

4\. Consider disabling SMBv1 entirely if not already

5\. Tune Wazuh alerting thresholds/notifications so rule 60204-level

&#x20;  alerts trigger real-time analyst notification, not just logging



\## Evidence

\- \[ ] Screenshot: netexec output showing successful credential (vboxuser:kali)

\- \[ ] Screenshot: Wazuh events list — failed logon cluster

\- \[ ] Screenshot: Wazuh correlation alert (rule 60204)

\- \[ ] Screenshot: Wazuh 4624 successful logon event (pending)

