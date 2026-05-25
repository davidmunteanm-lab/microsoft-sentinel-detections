# MITRE ATT&CK Mapping

Every detection and hunting query in this repo is mapped to a MITRE ATT&CK tactic and technique. This table is the index — open the linked file to see the full implementation, tuning notes, and response guidance.

## Detections

| File | Tactic | Technique | Description |
| --- | --- | --- | --- |
| [01-failed-logon-brute-force](../detections/01-failed-logon-brute-force.kql) | Credential Access | [T1110.001](https://attack.mitre.org/techniques/T1110/001/) Password Guessing | High-volume failed logons from a single source IP. |
| [02-successful-logon-after-brute-force](../detections/02-successful-logon-after-brute-force.kql) | Credential Access | [T1110](https://attack.mitre.org/techniques/T1110/) Brute Force (success) | Failure burst followed by a success on the same target — confirmed credential compromise. |
| [03-impossible-travel](../detections/03-impossible-travel.kql) | Initial Access | [T1078](https://attack.mitre.org/techniques/T1078/) Valid Accounts | Sign-ins from two locations farther apart than commercial-aviation speed allows. |
| [04-suspicious-powershell-execution](../detections/04-suspicious-powershell-execution.kql) | Execution / Defense Evasion | [T1059.001](https://attack.mitre.org/techniques/T1059/001/) PowerShell + [T1027](https://attack.mitre.org/techniques/T1027/) Obfuscation | PowerShell invoked with encoded commands, hidden windows, policy bypass. |
| [05-mass-file-deletion](../detections/05-mass-file-deletion.kql) | Impact | [T1485](https://attack.mitre.org/techniques/T1485/) Data Destruction + [T1486](https://attack.mitre.org/techniques/T1486/) Data Encrypted for Impact | High-volume file deletions in a short window — ransomware/wiper signature. |
| [06-mfa-fatigue-attack](../detections/06-mfa-fatigue-attack.kql) | Credential Access | [T1621](https://attack.mitre.org/techniques/T1621/) MFA Request Generation | Repeated MFA push prompts for a single user. |
| [07-suspicious-inbox-forwarding-rule](../detections/07-suspicious-inbox-forwarding-rule.kql) | Collection / Exfiltration | [T1114.003](https://attack.mitre.org/techniques/T1114/003/) Email Forwarding Rule | Mailbox rule created that auto-forwards or hides messages — BEC persistence pattern. |
| [08-privileged-group-addition](../detections/08-privileged-group-addition.kql) | Privilege Escalation / Persistence | [T1098](https://attack.mitre.org/techniques/T1098/) Account Manipulation + [T1078.002](https://attack.mitre.org/techniques/T1078/002/) Domain Accounts | User added to Domain Admins, Global Administrator, or other tier-0 group. |

## Hunting Queries

| File | Tactic | Technique | Description |
| --- | --- | --- | --- |
| [01-rare-process-execution](../hunting-queries/01-rare-process-execution.kql) | Execution / Defense Evasion | [T1059](https://attack.mitre.org/techniques/T1059/) Command Interpreter | Surfaces processes seen on few devices and rarely executed — new tooling indicator. |
| [02-dns-tunneling-indicators](../hunting-queries/02-dns-tunneling-indicators.kql) | Command and Control | [T1071.004](https://attack.mitre.org/techniques/T1071/004/) DNS + [T1048.003](https://attack.mitre.org/techniques/T1048/003/) Exfil over Other Protocol | Long DNS labels, high query volume to single domain — covert tunnel signature. |
| [03-lateral-movement-rdp](../hunting-queries/03-lateral-movement-rdp.kql) | Lateral Movement | [T1021.001](https://attack.mitre.org/techniques/T1021/001/) Remote Desktop Protocol | Accounts logging into many hosts via RDP in a short window. |
| [04-anomalous-cloud-app-usage](../hunting-queries/04-anomalous-cloud-app-usage.kql) | Initial Access / Defense Evasion | [T1078.004](https://attack.mitre.org/techniques/T1078/004/) Cloud Accounts + [T1550](https://attack.mitre.org/techniques/T1550/) Alternate Auth | Sign-ins from never-seen apps, countries, or devices vs. 30-day baseline. |

## Coverage summary by tactic

| Tactic | # of Queries |
| --- | --- |
| Initial Access | 2 |
| Execution | 2 |
| Persistence | 1 |
| Privilege Escalation | 1 |
| Defense Evasion | 3 |
| Credential Access | 3 |
| Lateral Movement | 1 |
| Collection | 1 |
| Command and Control | 1 |
| Exfiltration | 1 |
| Impact | 1 |

The coverage skews toward identity-and-endpoint, which is the most common SC-900-aligned scenario: Microsoft Entra ID + Defender for Endpoint + Microsoft 365 logs.

## References

- [MITRE ATT&CK Enterprise Matrix](https://attack.mitre.org/matrices/enterprise/)
- [DeTT&CT — mapping detections to ATT&CK](https://github.com/rabobank-cdc/DeTTECT)