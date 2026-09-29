# SOC Lab: Wazuh SIEM Detection Project

A hands-on Security Operations Center lab built to demonstrate SIEM deployment,
endpoint monitoring, and incident detection/investigation.

## Architecture

- **Ubuntu (192.168.100.10)** — Wazuh manager, indexer, and dashboard
- **Windows 10 (192.168.100.30)** — Monitored endpoint, Wazuh agent + Sysmon
- **Kali Linux (192.168.100.20)** — Attacker machine
- Isolated internal network ("LAB-NET") for inter-VM traffic, separate NAT adapters for internet access

## What this lab demonstrates

- Deploying and configuring a Wazuh SIEM stack from scratch
- Installing and centrally configuring Sysmon for deep endpoint telemetry
- Simulating a real attack (SMB brute-force) and investigating it end-to-end
- Correlating raw Windows Security events into a detection narrative
- Writing a client-ready incident report

## Contents

- `/incidents` — Investigated attack scenarios, written up as formal incident reports

## Skills demonstrated

Wazuh, Sysmon, Windows Event Log analysis, SMB protocol, netexec/hydra,
network segmentation, log correlation, MITRE ATT&CK mapping, incident documentation
