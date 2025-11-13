# 🚀 **Automated AlertFlow — Enrichment & Response with n8n + Wazuh**

A practical SOC automation project that builds an end-to-end alert enrichment, response, and reporting pipeline using **Wazuh**, **n8n**, and **VirusTotal**.
Designed to reduce analyst workload and shrink investigation time from **10+ minutes to under 3 minutes**, this project simulates a real Tier-1/Tier-2 workflow with automated triage and response.

---

## 🧭 **Table of Contents**

| Section                                                            | Description                                                           |
| ------------------------------------------------------------------ | --------------------------------------------------------------------- |
| **[1️⃣ Project Overview](./Project%20Overview/README.md)**         | Purpose, scope, and what this automation solves                       |
| **[2️⃣ Architecture](./Architecture/README.md)**                 | High-level workflow diagram & flow explanation                        |
| **[3️⃣ Deployment](./Deployment/README.md)**                     | How the environment was set up (Docker, n8n, forwarding, credentials) |
| **[4️⃣ Workflows](./Workflows/README.md)**                       | Breakdown of enrichment nodes, decision logic, report generation      |
| **[5️⃣ Scenario Walkthrough](./Scenario_Walkthrough/README.md)** | Full malicious + clean file simulations with end-to-end results       |

---

## 🎯 **Project Summary**

This project automates the entire lifecycle of a security alert:

**Wazuh detection → Forwarder script → n8n workflow → VirusTotal enrichment → Automated decision → Quarantine → SOC email report.**

The goal was to replicate a realistic SOC automation use-case and drastically reduce manual triage time by orchestrating a clean, low-code response system.

---

## ⭐ **Repo Highlights**

* 🔗 **Built a complete enrichment pipeline** integrating Wazuh → n8n → VirusTotal API
* 🧪 **Achieved dual-path logic for malicious vs. clean files** (two different report types)
* 🗂️ **Designed automated HTML SOC reports** with all key triage fields
* 🖥️ **Implemented automated quarantine on Windows endpoints** using n8n → SSH
* ⚠️ **Simulated realistic SOC scenarios** (file addition, malicious file detection, clean-file triage)
* 🔄 **Cut down manual investigation time from ~10 mins to <3 mins** through automation
* 🧠 **Designed workflow logic similar to real SOAR platforms** (enrichment → decision → action → reporting)

---

## 🧠 **Key Takeaways (What I Learned)**

Working on this project gave me hands-on experience with:

* Building **SOAR-like automation workflows** from scratch
* Performing **API integration** (VirusTotal), handling JSON parsing, and logic design
* Understanding the role of **enrichment in real SOC triage**
* Debugging n8n workflows and forwarder scripts like a real SOC engineer
* Designing **automated response actions** (quarantine, clean-file handling, reporting)
* Structuring end-to-end detection → enrichment → action pipelines

---

## 🤖 **Role of AI in This Project**

I used **ChatGPT and Claude** throughout the build to troubleshoot errors, design workflow logic, and correctly parse data structures in n8n.
AI significantly accelerated development time and helped solve complex debugging issues that normally require advanced experience.

---

## 🧩 **System Architecture (Placeholder for Diagram)**

*A clean architecture diagram will be placed here showing:
Wazuh → Forwarder → n8n → VirusTotal → Decision → Response → Email Reporting.*
---
<img width="832" height="490" alt="Screenshot 2025-11-13 235428" src="https://github.com/user-attachments/assets/82df7f73-8c47-4aee-8192-1369a37ab5e0" />

---

## 📁 **Project Structure**

```text
/AlertFlow-n8n/
│
├── 1_Project_Overview/
├── 2_Architecture/
├── 3_Deployment/
├── 4_Workflows/
├── 5_Scenario_Walkthrough/
└── README.md   ← (this file)
```

---

## 🧠 **Key SOC Metrics Achieved**

| Task                                      | Manual Time           | Automated Time         | Improvement         |
| ----------------------------------------- | --------------------- | ---------------------- | ------------------- |
| File investigation (hash lookup + triage) | ~10 minutes           | ~2–3 minutes           | **70% faster**      |
| Malicious file containment                | Manual steps required | Fully automated        | **Instant**         |
| Clean-file validation                     | Manual review         | Automated clean-report | **0 noise for SOC** |

---

## 🏁 **Future Enhancements (Simple Roadmap)**

* Add more detection scenarios (network alerts, privilege escalation, process anomalies)
* Expand enrichment (AbuseIPDB, WHOIS, URLScan)
* Improve HTML SOC report formatting
* Integrate Slack/Teams notifications

---

## 📬 **Connect With Me**

👉 **LinkedIn:** [https://www.linkedin.com/in/gurudayal-cybersecurity/](https://www.linkedin.com/in/gurudayal-cybersecurity/)

---

