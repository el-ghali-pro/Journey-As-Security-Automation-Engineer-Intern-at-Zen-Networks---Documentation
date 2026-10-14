# Scenario 1: SSH Bruteforce - Internal Threat Detection

## 📌 Overview

This scenario simulates a **dictionary-based SSH bruteforce attack** originating from within the internal network. The attack is detected by Zabbix monitoring the authentication logs, triggering the SOAR pipeline to create a Jira ticket and notify the SOC team via Slack. **No automatic blocking** is applied to prevent self-inflicted DoS on internal infrastructure.

**Threat Type:** Internal (Private IP)  
**Detection Method:** Zabbix Agent monitoring `/var/log/auth.log`  
**Response:** Passive (Manual investigation required)  
**Response Time:** ~6 seconds  

---

## 🎯 Attack Details

| Attribute | Value |
|-----------|-------|
| **Attack Tool** | Hydra |
| **Attack Vector** | SSH Service (Port 22) |
| **Attacker IP** | `192.168.20.152` (Kali Linux) |
| **Target IP** | `192.168.20.151` (Ubuntu Server) |
| **Target Hostname** | `ubuntu-client1` |
| **Dictionary** | `rockyou.txt` |
| **Threshold** | >5 failed attempts in 10 seconds |
| **Attack Command** | `hydra -l client1 -P /usr/share/wordlists/rockyou.txt -t 4 ssh://192.168.20.151` |

---

## 🔧 Zabbix Configuration

### Item: Capture malicious IP from /var/log/auth.log
<img width="1111" height="595" alt="Zabbix Item Configuration - Auth Log Monitoring" src="https://github.com/user-attachments/assets/17302539-71b4-408f-80fa-03945230024b" />


**Item Configuration:**
- **Type:** Zabbix Agent (Active)
- **Key:** `logrt` with RegEx pattern
- **Purpose:** Extract attacker IP from authentication logs
- **Update Interval:** 1 second

### Trigger: SSH-Bruteforce Detected
<img width="1201" height="880" alt="Zabbix Trigger Configuration - SSH Bruteforce" src="https://github.com/user-attachments/assets/8a21b3cb-578b-4fe0-a874-8775bcacc6fa" />


**Trigger Logic:**
```javascript
count(//var/log/auth.log, "Failed password", "10s") > 5
```

**Trigger Properties:**
- **Name:** SSH-Bruteforce Detected
- **Severity:** High
- **Function:** `count()` with RegEx pattern
- **Condition:** >5 failures in 10 seconds

---

## 💻 Attack Simulation

### Execution from Kali Linux
<img width="1033" height="303" alt="Kali Linux - Hydra SSH Bruteforce Attack" src="https://github.com/user-attachments/assets/c1e01fcc-ca17-4740-b06d-55c2dad29e34" />


```bash
hydra -l client1 -P /usr/share/wordlists/rockyou.txt -t 4 ssh://192.168.20.151
```

**Command Breakdown:**
- `-l client1`: Username to test
- `-P rockyou.txt`: Password dictionary file
- `-t 4`: 4 parallel connections
- `ssh://192.168.20.151`: Target SSH service

---

## 🔄 SOAR Pipeline Execution

### 1. Alert Generation
<img width="1282" height="816" alt="Zabbix Dashboard - Problem Generation" src="https://github.com/user-attachments/assets/6dc75a4b-e3b1-4ce7-b60f-74d2992daef6" />



The alert appears in the Problems Dashboard with **High** severity, confirming the SSH-Bruteforce detection on host `ubuntu-client1`.

### 2. Webhook Transmission
<img width="1424" height="747" alt="Zabbix Action Log - Webhook Sent" src="https://github.com/user-attachments/assets/8f163a48-0a46-4881-8517-18b1657817a6" />


**Action Log Details:**
- **Status:** Sent ✓
- **Type:** Shuffle Media
- **Event ID:** Captured from trigger
- **Attacker IP:** `192.168.20.152` (Internal)
- **MITRE Link:** Included in payload

### 3. Shuffle Workflow Execution
<img width="999" height="658" alt="Shuffle Workflow - Internal Threat Processing" src="https://github.com/user-attachments/assets/24dfdf15-9ce7-4478-bb51-f6896deb313c" />


**Workflow Path:**
1. Entry Node (Webhook) → Received ✓
2. Python ParseToJSON → Data normalized ✓
3. Jira 4 - Confirm Private IP → Branch selected ✓
4. Slack 4 → Notification sent ✓

**Key Observation:** The workflow bypasses pfSense blocking nodes (upper branch) since the IP is classified as internal.

### 4. Execution Status
<img width="934" height="339" alt="Shuffle - Execution History" src="https://github.com/user-attachments/assets/29cbd9f2-1670-42cb-bd7f-9bf1f2073e87" />


**Execution Metrics:**
- **Status:** FINISHED ✓
- **Start Time:** 19:55:32
- **End Time:** 19:56:25
- **Total Duration:** ~53 seconds (includes human interaction time)

### 5. Raw Payload Reception
<img width="571" height="313" alt="Shuffle - Webhook Payload" src="https://github.com/user-attachments/assets/cb83fbf8-4af5-4c89-817e-a6e477e336e4" />


**Payload Contains:**
- Subject: Problem: SSH-Bruteforce
- CPU Load: 58.69%
- Network traffic metrics
- MITRE ATT&CK URL
- Complete alert context

### 6. Data Normalization
<img width="577" height="417" alt="Shuffle - Python Parse Output" src="https://github.com/user-attachments/assets/35f90e38-5296-4327-8761-c03831be8de8" />


**Parsed Data (13 items):**
```json
{
  "event_id": "1270",
  "problem_name": "SSH-Bruteforce",
  "time": "19:55:31",
  "date": "2026.06.18",
  "severity": "High",
  "ioc": "192.168.20.152",
  "host_name": "ubuntu-client1",
  "host_id": "10684",
  "host_ip": "192.168.20.151",
  "cpu_usage": "58.69",
  "inbound_traffic": "2784",
  "outbound_traffic": "3904",
  "info_url": "https://attack.mitre.org/techniques/T1110/"
}
```

---

## 📋 Jira Ticket Creation

### API Response
<img width="477" height="274" alt="Jira API - Ticket Created" src="https://github.com/user-attachments/assets/c376417a-c024-4c0d-8d20-4616d13c2962" />



**Response Details:**
- **Status:** 201 Created

### Jira Ticket View
<img width="508" height="331" alt="Jira Ticket - Internal SSH Threat" src="https://github.com/user-attachments/assets/921b265a-cb1b-4105-8905-0559e6a6ac2b" />


**Ticket Details:**
- **Title:** [INTERNAL THREAT] SSH-Bruteforce
- **Priority:** High
- **Description includes:**
  - Event ID
  - Date/Time
  - CPU and network metrics
  - MITRE ATT&CK reference
  - Standardized investigation actions

---

## 💬 Slack Notification

### API Response
<img width="452" height="328" alt="Slack API - Notification Delivered" src="https://github.com/user-attachments/assets/7090aa69-4fa7-495f-8c6f-a7421f5658c7" />



### Slack Alert Card
<img width="515" height="193" alt="Slack - Internal Threat Notification Card" src="https://github.com/user-attachments/assets/5af25dbc-bfc3-4b83-b478-208a4ae5a2b3" />


**Notification Content:**
- **Title:** Zabbix Alert - Intern Threat
- **IP:** `192.168.20.152` (Internal Network)
- **Host:** `ubuntu-client1` (`192.168.20.151`)
- **Alert:** SSH-Bruteforce at 19:55:31
- **MITRE Link:** Direct URL to T1110
- **Jira Ticket:** with direct link
- **Action Required:** Manual investigation

---

## 📊 Summary

| Step | Component | Status | Output |
|------|-----------|--------|--------|
| 1. Detection | Zabbix Trigger | ✅ Success | Alert generated |
| 2. Ingestion | Webhook (Zabbix → Shuffle) | ✅ Success | Payload sent |
| 3. Normalization | Python ParseToJSON | ✅ Success | JSON structured |
| 4. Classification | IP Type Detection | ✅ Success | Identified as Internal |
| 5. Ticketing | Jira API | ✅ Success | Ticket created |
| 6. Notification | Slack API | ✅ Success | Alert sent to SOC team |
| 7. Blocking | pfSense | ⛔ Skipped | No blocking (internal IP) |

---

## 🎯 Key Takeaways

1. **Context-Aware Response:** The SOAR correctly identified the IP as internal and avoided automatic blocking that could cause self-DoS.

2. **Comprehensive Triage:** The Jira ticket contains all relevant context for SOC analysts to begin investigation immediately.

3. **Rapid Notification:** The SOC team is alerted in ~6 seconds, significantly faster than manual notification.

4. **MITRE Integration:** The alert includes direct links to ATT&CK techniques for faster threat understanding.

5. **Analyst Empowerment:** Rather than replacing human decision-making, the SOAR augments it by handling repetitive tasks and providing rich context.

---

## 🔗 Related Resources

- [MITRE ATT&CK T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/)
- [THC-Hydra Documentation](https://github.com/vanhauser-thc/thc-hydra)
- [Zabbix Log Monitoring Documentation](https://www.zabbix.com/documentation/current/en/manual/config/items/itemtypes/log_items)

---

*This scenario demonstrates the effectiveness of the SOAR pipeline for internal threats, prioritizing investigation and traceability over aggressive automated blocking.*
