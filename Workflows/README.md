# 🔄 Workflows — How I Designed & Built the Automation Logic

This section highlights the core workflow I designed in **n8n** to automate alert enrichment and response actions.  
Rather than documenting how someone else can rebuild it, this page focuses on **my decision-making, challenges, and what I learned** while developing the pipeline.

---

## 🎯 Purpose of the Workflow

The goal behind this workflow was simple:  
**automate malicious file triage so I could understand enrichment, reduce manual work, and get real hands-on SOC experience.**

As a beginner, I wanted to see how a raw alert moves through enrichment → decision → response, and building this helped me understand the full chain end-to-end.

---

## 🧩 High-Level Workflow Logic

I designed a multi-stage workflow with these core components:

Webhook → Extract Alert Data → VirusTotal Enrichment → Decision Check →
├── Malicious → Quarantine + SOC Report Email
└── Clean → Clean File Report Email

This structure helped me understand how real SOAR systems automate triage and decision-making.

---

## 🚧 What Was Challenging (And What I Learned)

When I started, **every part was new** — but a few things pushed me the most:

### **1. JSON Extraction & Parsing**  
I learned how to extract exactly the fields I needed from the Wazuh alert payload  
(e.g., file name, SHA256 hash, rule description, timestamp).

### **2. Decision Logic**  
Building the VirusTotal score evaluation taught me how enrichment actually influences SOC decisions.

### **3. HTML Report Generation**  
Formatting the final SOC report in HTML pushed me to understand how variables flow through nodes and how UI-based reporting works.

This hands-on debugging taught me how an automation engine interprets input/output between chained nodes.

---

## 🧠 Key Learning Outcomes

By building this workflow myself, I strengthened:

- **API integration skills** (VirusTotal)
- **JSON parsing and field extraction**
- **Automation logic design**
- **Alert → Enrichment → Response pipelines**
- **Using low-code tools to orchestrate SOC processes**

These are foundational skills used in SIEM/SOAR environments.

---

## 📨 The Auto-Generated SOC Report

Instead of dumping raw logs, I created an HTML-based SOC report that automatically fills key details pulled from the alert and enrichment:

- `file_name`
- `file_path`
- `file_sha256`
- `rule_id`
- `rule_description`
- `timestamp`
- `vt_score`
- `action_taken` (Quarantined / Clean)

**This makes the output understandable even without reading raw JSON**, which is exactly what a Tier-1 SOC workflow aims to achieve.

---

## 🧪 How I Validated the Workflow

To make sure my logic worked correctly:

1. Triggered a real alert from Wazuh  
2. Verified that VirusTotal API received the hash  
3. Confirmed whether the workflow quarantined the malicious file  
4. Checked email inbox for the SOC report

Validating it end-to-end helped me truly understand detection → enrichment → response.

---

## 🖼️ Screenshots (Backend Views of Nodes)

These screenshots show the **backend configuration** of the workflow nodes — not just the canvas — to demonstrate my understanding of input/output handling:

| Screenshot | Description |
|-----------|-------------|
| `workflow-canvas.png` | <img width="1712" height="507" alt="Screenshot 2025-11-13 152359" src="https://github.com/user-attachments/assets/276f0e64-1b0b-4615-90b5-a59a69577b05" />
Full pipeline as seen on the n8n canvas |
| `virustotal-node-backend.png` | <img width="1864" height="962" alt="Screenshot 2025-11-13 152107" src="https://github.com/user-attachments/assets/48808b9e-7f3b-466e-81de-c14381b9289a" />
VirusTotal API node showing inputs/outputs & enrichment mapping |
| `email-report-backend.png` | <img width="1847" height="944" alt="Screenshot 2025-11-13 152151" src="https://github.com/user-attachments/assets/97a166b2-6773-4d4c-ae73-39eb24c2139e" />
HTML email template containing variables extracted & enriched |

> *(I will add these sanitized backend screenshots for credibility.)*

---

## 🔍 Why This Workflow Matters

This workflow wasn’t about copying or re-creating a template — it was my way of learning:

- how alerts move through an automation pipeline,  
- how enrichment adds context to decisions,  
- and how response logic gets applied in real SOC operations.

It represents my first hands-on step into **SOAR-style automation**.

---

