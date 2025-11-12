# 2. Architecture — AlertFlow System 

This architecture page briefly explains the core automated workflow that handles suspicious file alerts: from Wazuh → n8n enrichment → decision → action → reporting.  
This diagram represents the n8n workflow for file enrichment + response
## n8n Workflow — File Enrichment & Response (Mermaid)

This diagram shows the automated workflow: **Wazuh alert → n8n enrichment (VirusTotal) → decision → action → reporting**.

```mermaid
flowchart LR
  %% Direction: Left to Right
  classDef box fill:#0b1220,stroke:#2b7cff,color:#ffffff,font-size:12px;
  classDef action fill:#0b2f14,stroke:#27ae60,color:#ffffff,font-size:12px;
  classDef email fill:#2b1b3a,stroke:#f39c12,color:#ffffff,font-size:12px;
  classDef process fill:#1b2430,stroke:#8aa4ff,color:#ffffff,font-size:12px;
  classDef decision fill:#2b2b2b,stroke:#ff6b6b,color:#ffffff,font-size:12px;

  subgraph intake [ ]
    direction LR
    WZ[/"Webhook (Wazuh alert)"/]:::box --> EX[/"Extract Alert Data\n(parse JSON, file hash)"/]:::box
  end

  EX --> VT[/"VirusTotal Scan\n(hash lookup API)"/]:::process

  VT --> DEC{VT Score >= 35\n(out of 67)?}:::decision

  %% Malicious branch
  DEC -- Yes --> QUAR[/"Quarantine File\n(move to quarantine folder)\n(execute generic command)"/]:::action
  QUAR --> REP["Prepare SOC Report\n(alert_id, hash, vt_score, context)"]:::process
  REP --> EMAIL[/"Send Alert Email\n(Gmail / SMTP)"/]:::email

  %% Clean branch
  DEC -- No --> CLEAN["Process Clean File\n(log / mark / notify)"]:::process
  CLEAN --> CLEANREP[/"Send Clean File Report\n(Gmail / SMTP)"/]:::email

  %% Notes / legend anchor
  class WZ,EX,VT,DEC,QUAR,REP,CLEAN,CLEANREP,EMAIL box;

Legend:
- Wazuh alerts are posted to the n8n Webhook (left).
- n8n extracts alert data and queries VirusTotal (VT).
- Decision node: VT score >= 35 (of 67) => treat as MALICIOUS (quarantine + alert).
- Else => treat as CLEAN (process & send clean-file report).
- SOC Report includes fields: alert_id, timestamp, host, file_path, file_hash,
  virustotal_score, vt_positive_names, src_ip/dest_ip, rule_id, action_taken, notes.

Forwarder:
- Alerts forwarded from Wazuh by `/usr/local/bin/wazuh_to_n8n.py` (tails alerts.json).

---

## How this maps into the system
- **Detection source:** Wazuh Manager generates alerts (forwarded by `/usr/local/bin/wazuh_to_n8n.py` to the n8n webhook).  
- **Orchestration:** n8n receives the alert (Webhook) → extracts key fields → queries VirusTotal → branches on result → executes remediation or logs a clean-file workflow → sends report email.  
- **Reporting:** Gmail (SMTP) used as a delivery channel for SOC summary/alerts (generic configuration).

---

## SOC Report — fields included (used to triage & confirm)
The `Prepare SOC Report` step includes critical fields for analyst triage:
- `alert_id` (Wazuh)  
- `timestamp` (first_seen / last_seen)  
- `host` (anonymized hostname)  
- `agent_os` (Windows / Ubuntu)  
- `file_path` (if applicable)  
- `file_hash` (MD5 / SHA256)  
- `virustotal_score` (positive_count / total_scans)  
- `vt_positive_names` (vendors flagging)  
- `src_ip` / `dest_ip` (network context)  
- `rule_id` (Wazuh rule)  
- `action_taken` (quarantined / ignored / investigated)  
- `investigator_note` (auto-populated template + manual notes)

---

## Success / Error Decision Logic (simple rule)
- **Success (malicious):** VirusTotal enrichment returns score **≥ 35** (out of 67) → treat as malicious → quarantine + alert.  
- **Error (clean):** VirusTotal score < 35 → treat as clean → process and send clean-file report.

> Note: Thresholds are lab-configurable. In production, tune by vendor votes/VT heuristic and other telemetry.

---

## Wazuh → n8n Forwarder
- Forwarder script: `/usr/local/bin/wazuh_to_n8n.py` (tails `/var/ossec/logs/alerts/alerts.json`, filters by rule IDs, posts to n8n webhook).  
- Run it under systemd (recommended) for persistence and auto-restart.

---

