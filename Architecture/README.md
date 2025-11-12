# 2. Architecture — AlertFlow System (ASCII Diagram)

This architecture page briefly explains the core automated workflow that handles suspicious file alerts: from Wazuh → n8n enrichment → decision → action → reporting.  
The diagram below represents the **n8n workflow** used for file enrichment + response (ASCII art, left → right).

Webhook (Wazuh alert) ──▶ Extract Alert Data ──▶ VirusTotal Scan
│ │
(parsed JSON) ┌───────┴───────┐
│ │
[If Success = Malicious] [If Error = Clean]
│ │
┌───────────────────────────┴───────────────┐ │
│ │ ▼
Quarantine File (move to quarantine folder) Process Clean File
(execute generic command) (mark / log / notify)
│ │
▼ ▼
Prepare SOC Report ──▶ Send Alert Email Send Clean File Report
(triage fields) (Gmail / SMTP) (Gmail / SMTP)

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

## MITRE ATT&CK mapping (brief)
This automated flow helps detect & respond to tactics including:
- **Execution** — T1204 (User Execution / malicious file execution)  
- **Credential Access** — T1110 (Brute Force) — relevant for other workflows (SSH)  
- **Discovery / Reconnaissance** — T1046 (Network Service Discovery) — when Suricata triggers are involved  
- **Defense Evasion** — T1070 (Indicator Removal on Host) — monitored via FIM

Include these mappings in alert metadata to assist triage and classification.

---
