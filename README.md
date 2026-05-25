# Microsoft Sentinel Detections

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![KQL](https://img.shields.io/badge/Language-KQL-blue.svg)](https://learn.microsoft.com/azure/data-explorer/kusto/query/)
[![MITRE ATT&CK](https://img.shields.io/badge/Mapped%20to-MITRE%20ATT%26CK-red.svg)](https://attack.mitre.org/)

A collection of detection rules and hunting queries written in **Kusto Query Language (KQL)** for Microsoft Sentinel. Every query is mapped to a MITRE ATT&CK technique, comes with tuning notes, and is ready to drop into a Sentinel workspace as a scheduled analytics rule or a saved hunting query.

Built as a companion to my [SOC Analyst Portfolio](https://github.com/davidmunteanm-lab/soc-analyst-portfolio) and as hands-on practice while preparing for the Microsoft SC-900 certification.

## What's inside

### `detections/` — Scheduled analytics rules

8 detection queries designed to be deployed as Sentinel analytics rules. Each rule has a severity, tactic, technique, expected data source, and tuning notes inline.

| # | Detection | Tactic | Severity |
| --- | --- | --- | --- |
| 01 | [Failed Logon Brute Force](detections/01-failed-logon-brute-force.kql) | Credential Access | Medium |
| 02 | [Successful Logon After Brute Force](detections/02-successful-logon-after-brute-force.kql) | Credential Access | High |
| 03 | [Impossible Travel](detections/03-impossible-travel.kql) | Initial Access | High |
| 04 | [Suspicious PowerShell Execution](detections/04-suspicious-powershell-execution.kql) | Execution / Defense Evasion | High |
| 05 | [Mass File Deletion (Ransomware)](detections/05-mass-file-deletion.kql) | Impact | Critical |
| 06 | [MFA Fatigue / Push Bombing](detections/06-mfa-fatigue-attack.kql) | Credential Access | High |
| 07 | [Suspicious Inbox Forwarding Rule](detections/07-suspicious-inbox-forwarding-rule.kql) | Collection / Exfiltration | High |
| 08 | [Privileged Group Addition](detections/08-privileged-group-addition.kql) | Privilege Escalation / Persistence | High |

### `hunting-queries/` — Proactive threat hunts

4 hunting queries — broader, more exploratory queries meant to be run on demand during investigations or threat hunts, not turned into noisy alerts.

| # | Hunt | Tactic |
| --- | --- | --- |
| 01 | [Rare Process Execution](hunting-queries/01-rare-process-execution.kql) | Execution / Defense Evasion |
| 02 | [DNS Tunneling Indicators](hunting-queries/02-dns-tunneling-indicators.kql) | Command and Control / Exfiltration |
| 03 | [Lateral Movement via RDP](hunting-queries/03-lateral-movement-rdp.kql) | Lateral Movement |
| 04 | [Anomalous Cloud App Usage](hunting-queries/04-anomalous-cloud-app-usage.kql) | Initial Access / Defense Evasion |

### `docs/` — Reference material

- **[sentinel-architecture.md](docs/sentinel-architecture.md)** — How Sentinel works end-to-end (SC-900 level), the four pillars, alerts vs. incidents, when to pick Sentinel vs. a third-party SIEM.
- **[kql-cheatsheet.md](docs/kql-cheatsheet.md)** — Practical KQL patterns for the operators I actually use day-to-day.
- **[mitre-mapping.md](docs/mitre-mapping.md)** — Index of every query mapped to its MITRE ATT&CK technique.

## Data sources expected

These queries reference standard tables from the Microsoft Sentinel schema:

| Table | Source connector |
| --- | --- |
| `SecurityEvent` | Windows Security Events |
| `SigninLogs` | Microsoft Entra ID |
| `AuditLogs` | Microsoft Entra ID |
| `OfficeActivity` | Microsoft 365 / Exchange Online |
| `DeviceProcessEvents`, `DeviceFileEvents` | Microsoft Defender for Endpoint |
| `DnsEvents` | DNS data connector |

Every query declares its expected data source in the header comment.

## How to use

1. Open a Microsoft Sentinel workspace in the Azure portal.
2. **For analytics rules:** copy the KQL from a `detections/` file into **Analytics → Create → Scheduled query rule**. Configure schedule, severity, and entity mappings per the header comments.
3. **For hunting queries:** copy the KQL from a `hunting-queries/` file into **Hunting → Queries → New query**, or run it directly in Logs.
4. Tune the thresholds in `let` variables at the top of each query to match your environment's baseline. Most rules are intentionally conservative to keep the false-positive rate low in initial deployment.

## Design choices

A few conventions used across all queries:

- **Time filter first** — every query opens with a `where TimeGenerated >= ago(...)` to limit scan size.
- **`let` constants at the top** — thresholds, lookback windows, and whitelists are declared as variables so they're easy to tune without rewriting the query body.
- **Severity scoring with boolean math** — for detections where several factors combine into a verdict (e.g. suspicious PowerShell), each factor contributes points to a `SuspicionScore`. Tunable, transparent, easy to debug.
- **Tuning notes inline** — every query header explains what the rule catches, why it matters, and where false positives are likely. Aimed at the on-call analyst, not just the rule author.

## Validation

Queries were authored against the [documented Sentinel schema](https://learn.microsoft.com/azure/sentinel/data-source-schema-reference) and validated for KQL syntax with the official Microsoft Kusto VS Code extension. They reference real table and column names from the standard data connectors and are ready to run unchanged in a workspace with the appropriate connectors enabled.

## References

- [Microsoft Sentinel documentation](https://learn.microsoft.com/azure/sentinel/)
- [Azure Sentinel — official sample repository](https://github.com/Azure/Azure-Sentinel)
- [KQL quick reference](https://learn.microsoft.com/azure/data-explorer/kql-quick-reference)
- [MITRE ATT&CK Enterprise Matrix](https://attack.mitre.org/matrices/enterprise/)
- [SC-900 study guide](https://learn.microsoft.com/credentials/certifications/exams/sc-900/)

## Contact

- LinkedIn: [david-marian-muntean](https://www.linkedin.com/in/david-marian-muntean-a41313286/)
- Email: david.muntean.m@gmail.com