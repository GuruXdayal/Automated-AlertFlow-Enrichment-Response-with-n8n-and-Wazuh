# 🏗️ Architecture — AlertFlow System

This section explains the high-level architecture of the **AlertFlow Enrichment & Response System** I built using **Wazuh → n8n → VirusTotal → Email reporting**.  
Rather than showing how to reconstruct the workflow, this page focuses on **how the system fits together, why I designed it this way, and what I learned while building it.**

---

## 🔎 What This Architecture Represents

This architecture captures the exact logic I implemented in n8n for automated file enrichment and response.  
It shows how:

- A **Wazuh alert** enters the automation pipeline  
- n8n **extracts and enriches** the alert using VirusTotal  
- A **decision node** classifies the file as malicious or clean  
- The system generates a **SOC report** and sends it via email  
- A quarantine action is performed when required  

This diagram helped me understand the *end-to-end flow* of how SOAR-style automation works in real SOC environments.

---

## 🔧 High-Level Workflow Diagram 
<img width="774" height="514" alt="Screenshot 2025-11-13 154651" src="https://github.com/user-attachments/assets/a06cf425-5547-40ee-82fe-9e231c37fa75" />

---

## 🧠 How This Architecture Helped Me Understand SOC Automation

Designing this architecture helped me grasp several key SOC concepts:

---

### **1. Alert → Enrichment → Action Pipeline**

I learned how an alert evolves as it moves through a workflow:

- Raw alert enters through the webhook  
- Important fields are extracted  
- External enrichment adds context  
- Logic classifies the severity  
- Automated actions take place  

This mirrors the core principles of real SOAR design.

---

### **2. Decision-Making Based on Enrichment**

The VirusTotal score (≥ **35**) became my classification threshold.  
This taught me how **enrichment data directly influences triage decisions** and how SOC tools make contextual decisions automatically.

---

### **3. Response Actions & Reporting**

I implemented:

- **Quarantine logic** for malicious findings  
- **HTML-based SOC reports** containing key triage fields  
- **Automated email notifications** for visibility  

This helped me understand how SOC teams **communicate, escalate, and document alerts**.

---

## 🧵 Data Elements Used in the Pipeline

These are the key fields extracted, enriched, and included in the SOC report:

- `file_name`  
- `file_path`  
- `file_sha256`  
- `timestamp`  
- `rule_id`  
- `rule_description`  
- `vt_score`  
- `action_taken`  

This selection reflects what analysts typically use during **quick triage and validation**.

---

## 🔗 Where This Architecture Fits in the Whole Project

- **Wazuh** generates the alert  
- **Forwarder script** (`/usr/local/bin/wazuh_to_n8n.py`) sends it to n8n  
- **n8n** performs extraction, enrichment, decision-making, and reporting  
- **Gmail SMTP** delivers the final report to the analyst  

This architecture ties the entire workflow together and demonstrates how automation significantly reduces **manual investigation time**.

---

