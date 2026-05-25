# Microsoft Sentinel — Architecture Overview

Notes on how Microsoft Sentinel fits together, written at the level expected for SC-900 and useful as a quick reference when explaining the platform to a non-security audience.

## What Sentinel actually is

Microsoft Sentinel is a **cloud-native SIEM and SOAR**:

- **SIEM** — Security Information and Event Management. It collects logs from many sources, lets you query them, and generates alerts based on rules you define.
- **SOAR** — Security Orchestration, Automation, and Response. It can run automated playbooks in response to incidents (block an account, isolate a host, open a ticket).

Sentinel runs **on top of a Log Analytics workspace**, which is the actual storage and query layer. This is an important distinction: when you "deploy Sentinel," what you're really doing is enabling the Sentinel feature set on an existing Log Analytics workspace.

## The four pillars

Microsoft documents Sentinel around four functions, and the documentation, UI, and certification questions all reuse this framing.

| Pillar | What it covers |
| --- | --- |
| **Collect** | Data connectors (Microsoft 365, Entra ID, Defender XDR, AWS, GCP, Syslog, custom). Each connector ingests logs into the workspace. |
| **Detect** | Analytics rules — scheduled queries that run on a schedule and generate alerts when conditions are met. |
| **Investigate** | Incidents (groups of related alerts), the investigation graph, hunting queries, and notebooks. |
| **Respond** | Automation rules and playbooks (built on Azure Logic Apps) that take action when an incident is created. |

## Alerts vs. incidents

A common SC-900 distinction:

- **Alert** — a single signal. The output of one detection rule firing once.
- **Incident** — a case that groups one or more related alerts. Analysts work on incidents, not individual alerts. Sentinel can group alerts automatically based on entity (same user, same host) and time proximity.

## Key components in the UI

- **Data connectors** — where you onboard log sources.
- **Analytics** — where detection rules live. Three main types:
  - **Scheduled** — KQL queries that run on a timer (e.g., every 5 minutes).
  - **Microsoft Security** — rules that pass through alerts from Microsoft Defender products.
  - **Fusion** — Microsoft-managed machine-learning rules that correlate alerts from multiple sources.
- **Hunting** — saved KQL queries used for proactive threat hunting, not generating alerts directly.
- **Notebooks** — Jupyter notebooks integrated with the workspace, useful for complex investigations and ML.
- **Workbooks** — interactive dashboards (similar to Splunk dashboards).
- **Watchlists** — small datasets you can reference in KQL (e.g., list of executives, list of approved VPN IPs).
- **Threat intelligence** — IOCs from TI feeds, queryable as the `ThreatIntelligenceIndicator` table.

## How a detection turns into a response

The end-to-end flow most often described in SC-900 study material:

1. **Ingestion** — a log source (e.g., Microsoft Entra ID sign-ins) is connected via a data connector. Events stream into the Log Analytics workspace and land in standard tables (`SigninLogs`, `SecurityEvent`, `DeviceProcessEvents`, etc.).
2. **Detection** — an analytics rule runs on a schedule. The rule is a KQL query plus configuration (severity, MITRE tactics, entity mappings).
3. **Alert** — when the query returns results, Sentinel creates an alert.
4. **Incident** — automation rules can group related alerts into an incident and assign owner, severity, status.
5. **Response** — a playbook (Logic App) can run automatically: disable the user in Entra, isolate the device in Defender, post to a Teams channel.

This repo focuses on step 2 — the detection-rule KQL queries themselves.

## When Sentinel makes sense

A useful framing question: *why pick Sentinel over a third-party SIEM?*

Sentinel wins when:
- The organization is already mostly Microsoft (Entra ID, Microsoft 365, Defender XDR, Azure infrastructure). The connectors are native and free for first-party Microsoft data.
- You want to avoid log-egress costs from cloud services. Sending Microsoft logs to a third-party SIEM means paying twice (once to generate, once to ingest).
- You want SOAR built in (Logic Apps).
- You want UEBA, ML-based correlation (Fusion), and a managed update pipeline.

Sentinel is less obvious when the environment is mostly on-premises non-Microsoft, mostly AWS/GCP, or already invested in a different SIEM with mature content.

## How this repo fits

The KQL files in `detections/` and `hunting-queries/` are written to drop directly into a Microsoft Sentinel workspace as analytics rules or hunting queries. They reference standard table names (`SecurityEvent`, `SigninLogs`, `DeviceFileEvents`, etc.) from the documented Sentinel schema. Each file declares its expected data source in the header so it's clear which connector needs to be enabled to use it.

## References

- [Microsoft Sentinel documentation](https://learn.microsoft.com/azure/sentinel/)
- [Sentinel sample queries — Microsoft GitHub](https://github.com/Azure/Azure-Sentinel)
- [SC-900 study guide](https://learn.microsoft.com/credentials/certifications/exams/sc-900/)