# Scenario Walkthrough — End-to-End Alert Enrichment & Response

This section walks through the **exact real-world scenario** I automated end-to-end using Wazuh → n8n → VirusTotal → automated response → Gmail reporting.  
Rather than explaining how to recreate it, this section narrates **what happened**, **why**, and **what I learned** during the process.

The goal was to simulate a realistic security event, observe how my detection pipeline behaves, and validate the automation in action.

---

## 🎯 Scenario Simulated
To test the enrichment and automated response pipeline, I intentionally introduced a **test malware sample** on the Windows endpoint:

> **File:** `malicious.exe`  
> **Source:** Downloaded from EICAR test repository  
> **Path:** `C:\Users\Public\malicious.exe`  
> **Method:** Downloaded via PowerShell using `curl` into the Public directory

This created a credible file-based threat scenario for Wazuh to detect and n8n to triage.

---

## 🖼️ Screenshot 1 — File Created on Windows  
<img width="1575" height="831" alt="Screenshot 2025-11-13 195131" src="https://github.com/user-attachments/assets/1d70ca8b-c72c-4761-8c27-56a3a29ae69e" />

---

## 1️⃣ Wazuh Detects the File (Rule 554 — File Added)

As soon as `malicious.exe` appeared, **Wazuh’s File Integrity Monitoring** triggered:

- **Rule ID:** 554 (File Added)  
- **Rule Description:** *"File added to the system"*  
- **File Path:** `c:\\users\\public\\malicious.exe`  
- **File Name:** `malicious.exe`  
- **SHA-256:**  
  `275a021bbfb6489e54d471899f7db9d1663fc695ec2fe2a2c4538aabf651fd0f`

This was the raw detection signal that initiated the entire automation chain.

---

## 🖼️ Screenshot 2 — Wazuh Alert View  
<img width="1920" height="1200" alt="Screenshot 2025-11-13 195508" src="https://github.com/user-attachments/assets/4a1e57a8-30f9-4ca4-a09d-7cb9df7f2d0c" />

---

## 2️⃣ Forwarder Sends the Alert to n8n

My forwarder script (running under systemd) tailed `alerts.json`, matched the event against the configured rule, and forwarded a **sanitized, minimal payload** to the n8n webhook.

### Fields forwarded:
- `file_path`
- `file_name`
- `sha256`
- `rule_id`
- `rule_description`
- `agent_name`
- `timestamp`
- `alert_type`

This keeps the automation engine efficient and reduces noise by forwarding only essential indicators.

---

## 3️⃣ n8n Performs VirusTotal Enrichment

The forwarded hash was then used in n8n’s VirusTotal node to perform a live lookup.

### VirusTotal Result:
- **Score:** `66 / 69`  
- **Threat Names:** None (expected — this is a known EICAR test sample)  
- **Lookup Reliability:** First lookup failed due to formatting issues, worked after fixes  
- **Confidence:** Treated as malicious due to high detection ratio

This was my hands-on experience understanding how enrichment adds context to bare alerts.

---

## 🖼️ Screenshot 4 — VirusTotal Node Backend  
<img width="1864" height="962" alt="Screenshot 2025-11-13 152107" src="https://github.com/user-attachments/assets/0de0fe7e-cc81-4f05-97ee-8e1151228f62" />

---

## 4️⃣ Decision Logic — Is the File Malicious?

My automation evaluates:

> **If VirusTotal score ≥ 35 → treat as malicious**

Since the score was **66/69**, automation selected the **Malicious Branch (Yes)**.

This was the step where I learned how SOAR tools make structured decisions based on enrichment.

---

## 🖼️ Screenshot 5 — Decision Node Backend  
<img width="945" height="469" alt="Screenshot 2025-11-13 203415" src="https://github.com/user-attachments/assets/5128685e-0287-4057-a4da-0906394e7331" />

---

## 5️⃣ Automated Response — Quarantine via SSH

For malicious findings, my workflow automatically isolated the file by moving it into a quarantine directory on the Windows endpoint over SSH.

### **Action:**
The automation executed a PowerShell command: Move-Item -Path "C:\Users\Public\malicious.exe" -Destination "C:\Quarantine\malicious.exe"

### **Quarantine Path:**
`C:\Quarantine\`
<img width="346" height="231" alt="Screenshot 2025-11-13 203808" src="https://github.com/user-attachments/assets/22073978-4e97-4a51-90f7-1039a3fae18c" />


This step helped me understand how incident response tasks can be immediately triggered without analyst intervention.

---

## 🖼️ Screenshot 6 — SSH Quarantine Node Backend  
<img width="901" height="301" alt="Screenshot 2025-11-13 203633" src="https://github.com/user-attachments/assets/aec9418a-a51e-49e3-868a-6043ecb138b9" />

---

## 6️⃣ Automated SOC Report → Delivered to Gmail

The workflow then generated a clean, HTML-formatted SOC report summarizing the event:

### **Report Included:**
- **Alert ID:** 554  
- **Endpoint:** Windows-Agent  
- **File Path:** `c:\users\public\malicious1.exe`  
- **SHA-256:**  
  `275a021bbfb6489e54d471899f7db9d1663fc695ec2fe2a2c4538aabf651fd0f`  
- **Detection Time:** Nov 11, 2025, 08:43:33 PM UTC  
- **VirusTotal Score:** 66/69  
- **🛡️ Automated Response: *Successfully Quarantined***  
- **Quarantine Location:** `C:\Quarantine\malicious1.exe`

The report clearly explained what occurred and what action the automation took.

---

## 🖼️ Screenshot 7 — Gmail SOC Report  
<img width="381" height="435" alt="Screenshot 2025-11-13 204022" src="https://github.com/user-attachments/assets/26b5477a-7574-4a28-8e86-63ecec1b95b6" />

---

## 7️⃣ Non-Malicious File Walkthrough (Control Test)

This scenario validates how the automation behaves when the file is **benign** and **no malicious indicators** are detected.

### 📄 File Details
- **File:** `wazuh-n8n-test1.txt`  
- **Location:** saved on the Windows endpoint  
- **Purpose:** validate the “clean” branch of the workflow

### 🛎️ Wazuh Detection
Even though the file is harmless, Wazuh still triggers the **554 – File added** alert as part of FIM monitoring.  
This ensures the same forwarding pipeline is used for both malicious and non-malicious files.

---

## 🖼️ Screenshot 8 — Wazuh Alert View
<img width="570" height="532" alt="Screenshot 2025-11-13 225930" src="https://github.com/user-attachments/assets/f352474d-7df3-4416-99e6-12a058ed4295" />

---

### 9️⃣ VirusTotal Enrichment
- The VirusTotal node reported **“No hash detected”**  
  (expected behavior because `.txt` files have no executable signature)

Because enrichment returned **no malicious indicators**, the workflow correctly followed the *clean branch*.

### 🧠 Decision Logic Outcome
- **VT Score:** *N/A (hash missing)*  
- **Decision:** *Clean*  
- **Rule applied:** if VT score is **not available or below threshold**, classify as **non-malicious**.
---
### 🔟 Clean-File SOC Report (Email)
A separate SOC report is generated for clean files — confirming the system can handle safe files **without unnecessary remediation**.

**Key fields reported:**
- File Name: `wazuh-n8n-test1.txt`
- File Path: (as captured by Wazuh)
- Detection Time
- Rule ID: `554`
- VT Status: *No hash detected / No malicious indicators*
- Action Taken: **“No quarantine – file treated as benign”**
---

## 🖼️ Screenshot 9 — Gmail SOC Report 
<img width="343" height="464" alt="Screenshot 2025-11-13 230929" src="https://github.com/user-attachments/assets/f3fd8b3b-317d-402e-a0fe-a8db9db1f6cd" />

---

# 🧠 What This Simulation Taught Me

This scenario gave me practical insight into:

### **1. How real detections turn into structured, enriched signals**  
I saw firsthand how raw FIM alerts transform into actionable intelligence through VirusTotal enrichment.

### **2. How to design automation logic that mirrors real SOC workflows**  
Building the decision → quarantine → notifications chain taught me how orchestration tools reduce triage time.

### **3. How to validate end-to-end detection pipelines**  
Triggering the file, checking Wazuh, analyzing the forwarded payload, verifying enrichment, and receiving the final report gave me a complete understanding of the workflow lifecycle.

### **4. How to think like a SOC Analyst & Automation Engineer**  
This project showed me the analytical thinking behind:
- deciding thresholds  
- validating malicious indicators  
- choosing appropriate response actions  
- generating meaningful analyst-friendly reports  

---

# 📌 Summary of the Walkthrough

This scenario ties together everything I built in the project:

1. **Simulated a malicious file event on Windows**  
2. **Wazuh detected it instantly (Rule 554)**  
3. **Forwarder transmitted alert to n8n**  
4. **n8n enriched with VirusTotal**  
5. **Decision node classified file as malicious**  
6. **SSH node quarantined the file**  
7. **HTML SOC Report emailed to my inbox**

This is a complete, realistic demonstration of **SOC automation**, showing both detection and response capabilities.

---
