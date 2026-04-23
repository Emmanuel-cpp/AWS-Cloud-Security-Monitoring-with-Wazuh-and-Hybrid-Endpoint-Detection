# Building a Hybrid SOC — Wazuh SIEM Monitoring AWS EC2 Workloads and On-Premise Endpoints from a Single Cloud Dashboard

![Wazuh](https://img.shields.io/badge/Wazuh-4.7.5-blue?style=for-the-badge)
![AWS](https://img.shields.io/badge/AWS-Cloud-FF9900?style=for-the-badge&logo=amazonaws)
![MITRE](https://img.shields.io/badge/MITRE-ATT%26CK%20Mapped-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

---

## What This Project Is

Phase 3 of a three-part security monitoring series. A Wazuh SIEM server was deployed on AWS EC2 and configured to monitor cloud infrastructure and on-premise endpoints simultaneously — one dashboard, one alert pipeline, across two environments.

> Phase 1 — [OSSEC Host-Based IDS](https://github.com/Emmanuel-cpp/Host-Based-Intrusion-Detection-System-HIDS-Lab-OSSEC.git)
> Phase 2 — [Wazuh On-Premise SIEM](https://github.com/Emmanuel-cpp/Wazuh-Enterprise-Security-Lab.git)

---

## Architecture

```
AWS Cloud (us-east-1)
├── Wazuh Server EC2 — Ubuntu 22.04, t2.large (Manager + Indexer + Dashboard)
├── EC2 Linux Agent  — Ubuntu 22.04, t2.micro
├── EC2 Windows Agent — Windows Server 2022, t2.medium
├── CloudTrail → S3 → Wazuh AWS Module
├── GuardDuty — enabled
└── VPC Flow Logs — enabled

On-Premise
└── Windows 11 Host — Wazuh agent connecting outbound to AWS
```

All agents report to the cloud Wazuh server. On-premise agents connect outbound over the internet — no inbound port forwarding required.

---

## Lab Setup

![Agents List](screenshots/wazuh-agents-list.png)
> Three active agents across AWS and on-premise — EC2 Linux, EC2 Windows Server 2022, and on-premise Windows 11 — all running Wazuh 4.7.5 and reporting to the same cloud server.

![CloudTrail Active](screenshots/cloudtrail-active.png)
> CloudTrail trail active and logging. Event history already capturing VPC Flow Log creation — proof that AWS API activity is flowing into the monitoring pipeline.

---

## Attacks and Detections

### SSH Brute Force — EC2 Linux

An attacker EC2 instance inside the same VPC launched a full credential brute force against the Linux agent using Hydra and the rockyou wordlist.

![Hydra Attack](screenshots/hydra-attack-ec2.png)
> Hydra running from inside AWS — 14,344,398 password attempts against the EC2 Linux private IP.

![EC2 Linux Level 10](screenshots/ec2-linux-brute-force-level10.png)
> Rule 40111 Level 10 — Multiple authentication failures. MITRE T1110 Credential Access mapped automatically across all alerts.

---

### Privilege Escalation — EC2 Windows and On-Premise Windows

A backdoor user was created and added to the Administrators group on both the EC2 Windows instance and the on-premise Windows machine.

![EC2 Windows Level 12](screenshots/ec2-windows-level12-admin-changed.png)
> Rule 60154 Level 12 Critical — Administrators group changed on EC2 Windows. MITRE T1484 Defense Evasion and Privilege Escalation. Rule 60109 Level 8 — user account created, T1098 Persistence.

![Both Agents Level 12](screenshots/level12-both-windows-agents.png)
> The same attack detected on both agents simultaneously. Rule 60154 on ec2-windows-agent and Rule 18217 on windows-endpoint — cloud and on-premise alerts side by side on one dashboard.

---

### MITRE ATT&CK Coverage

![MITRE Overview](screenshots/wazuh-level12-mitre-overview.png)
> Wazuh automatically mapped detections across nine MITRE ATT&CK techniques including Valid Accounts, Account Manipulation, Domain Policy Modification, Pass the Hash, Remote Desktop Protocol, and Brute Force — with no manual tagging.

---

## Key Results

| Metric | Result |
|---|---|
| Highest alert level | 12 Critical — Administrators group changed |
| MITRE techniques auto-detected | 9 |
| Agents monitored | 3 active across 2 environments |
| AWS services integrated | CloudTrail, GuardDuty, VPC Flow Logs |
| On-premise connectivity | Outbound only — no port forwarding needed |

---

## Challenges Worth Noting

**CGNAT** — The local ISP uses Carrier Grade NAT making port forwarding impossible. The original plan was to run Wazuh on-premise and connect EC2 agents to it. CGNAT made this unworkable. The solution was moving the Wazuh server to AWS so all agents connect outbound — a cleaner architecture that mirrors how enterprise cloud SOCs are actually structured.

**EBS Disk Size** — The Wazuh dashboard installation failed because the default EC2 root volume was 8GB. The dashboard package alone requires close to 1GB of additional space. The volume was resized to 50GB and the partition expanded before the installer completed successfully.

**Agent Version Mismatch** — A pre-existing Wazuh agent on the EC2 Linux instance was version 4.14.4 while the server was 4.7.5. Wazuh rejects agents running newer versions than the manager. The agent was downgraded to match.

**CloudTrail Credentials** — The Wazuh AWS module could not locate the AWS credential profile regardless of where the credentials file was placed. Embedding the access key and secret key directly in ossec.conf resolved it immediately.

---

## AWS Native Security Tools

| Tool | What it covers | Status |
|---|---|---|
| CloudTrail | Every AWS API call | Integrated with Wazuh |
| GuardDuty | ML-based threat detection | Enabled |
| VPC Flow Logs | Network traffic metadata | Enabled |
| AWS Security Hub | Aggregates all findings | Next integration |
| Amazon Inspector | EC2 vulnerability scanning | Next integration |

Wazuh fills the gap these tools leave — visibility inside EC2 instances at the host level and unified monitoring across cloud and on-premise from one place.

---

## What Comes Next

Integrating AWS Security Hub to feed GuardDuty findings directly into Wazuh alerts. Adding Amazon Inspector for automated CVE scanning on the EC2 instances. Extending the same Wazuh server to monitor Azure — the foundation for a multi-cloud SOC.

---

## Project Series

- Phase 1 — OSSEC: Host-based intrusion detection from the raw engine
- Phase 2 — Wazuh On-Premise: Full SIEM with independent external endpoints
- Phase 3 — This project: Hybrid cloud SOC with AWS infrastructure and native security service integration

---

## Author

**Emmanuel Siamoonga**
Cloud Infrastructure | Network and Cloud Security | The Copperbelt University, Kitwe, Zambia

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=flat&logo=linkedin)](https://www.linkedin.com/in/emmanuel-siamoonga-98b30929b/)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black?style=flat&logo=github)](https://github.com/Emmanuel-cpp)

> "Security is not a product, but a process." — Bruce Schneier
