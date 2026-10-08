# Wazuh-_SIEM_FIM_lab
Deploying Wazuh SIEM on Ubuntu and configuring Real-Time File Integrity Monitoring (FIM) on a Windows Agent.

# Real-Time File Integrity Monitoring (FIM) with Wazuh SIEM

## 📌 Project Overview
This project demonstrates the deployment of a **Wazuh SIEM** environment to implement and validate **File Integrity Monitoring (FIM)** on a Windows endpoint in real time. 

The central manager collects and analyzes endpoint log activity, detecting critical file changes including creation, modification, and deletion events.

---

## 🏗️ Architecture & Lab Setup
* **Central Manager / Server:** Ubuntu Linux running Wazuh Manager (`v4.12.0`)
* **Monitored Endpoint:** Windows Host running Wazuh Agent (`v4.12.0`)
* **Network / Firewall:** Communication over ports `1514` (UDP/TCP) and `1515` (TCP)

---

## ⚙️ Configuration Steps

### 1. Enable FIM Directory Monitoring
Added target test path (`C:\wazuh_test`) to the `<syscheck>` block inside `ossec.conf` on the Windows agent:

```xml
<syscheck>
  <directories realtime="yes">C:\wazuh_test</directories>
</syscheck>
