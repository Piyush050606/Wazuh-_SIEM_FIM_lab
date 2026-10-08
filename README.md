# 🛡️ Real-Time File Integrity Monitoring (FIM) with Wazuh SIEM

![Wazuh](https://img.shields.io/badge/Wazuh-4.12.0-blue)
![Manager](https://img.shields.io/badge/Manager-Ubuntu%20Linux-E95420)
![Agent](https://img.shields.io/badge/Agent-Windows-0078D6)


## 📌 Overview

This project deploys a **Wazuh SIEM** environment to implement and validate **File Integrity Monitoring (FIM)** on a Windows endpoint in real time.

A central manager collects and analyzes endpoint activity, detecting critical file changes: **creation, modification, and deletion**.

## 📑 Table of Contents

- [Architecture & Network Setup](#-architecture--network-setup)
- [Installation & Deployment](#-installation--deployment)
- [FIM Configuration & Event Generation](#-fim-configuration--event-generation)
- [Verification & Results](#-verification--results)
- [Security Takeaways](#-security-takeaways)

---

## 🏗️ Architecture & Network Setup

| Role | System | Component | Version |
|------|--------|-----------|---------|
| Central Manager / Server | Ubuntu Linux | Wazuh Manager, Indexer, Dashboard | `4.12.0` |
| Monitored Endpoint | Windows Host | Wazuh Agent | `4.12.0` |

```text
┌─────────────────────┐      1514 (UDP/TCP) logs       ┌──────────────────────┐
│  Windows Endpoint   │ ─────────────────────────────▶ │   Ubuntu Server      │
│  Wazuh Agent        │      1515 (TCP) enrollment     │   Wazuh Manager      │
│  FIM: C:\wazuh_test │ ─────────────────────────────▶ │   Indexer + Dashboard│
└─────────────────────┘                                └──────────────────────┘
```

### Network Bridging & Connectivity

To enable communication between the host and manager across local networks:

- **Network adapter mode:** Set the virtual environment/network interfaces to **Bridged Mode**, giving both systems distinct IP addresses on the same subnet.
- **Firewall (Ubuntu):** Opened the following ports for agent registration and telemetry:

| Port | Protocol | Purpose |
|------|----------|---------|
| `1514` | UDP/TCP | Agent communication and log collection |
| `1515` | TCP | Agent enrollment and authentication |

---

## ⚙️ Installation & Deployment

### 1. Wazuh Manager Installation (Ubuntu)

Installed the Wazuh Manager, Indexer, and Dashboard using the official all-in-one installer:

```bash
# Update system repositories
sudo apt update && sudo apt upgrade -y

# Download and run the Wazuh installer
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

### 2. Wazuh Agent Deployment (Windows)

1. Downloaded and installed the matching **v4.12.0** Wazuh Agent MSI package.
2. Set the Ubuntu Manager's IP address in `C:\Program Files (x86)\ossec-agent\ossec.conf`:

```xml
<client>
  <server>
    <address>MANAGER_IP</address>
    <port>1514</port>
  </server>
</client>
```

3. Registered the agent with `agent-auth.exe` and confirmed **Active** status in the Wazuh dashboard.

---

## 🧪 FIM Configuration & Event Generation

### 1. Enable Directory Monitoring

Added the test path `C:\wazuh_test` to the `<syscheck>` block in the agent's `ossec.conf`:

```xml
<syscheck>
  <directories realtime="yes">C:\wazuh_test</directories>
</syscheck>
```

### 2. Restart the Wazuh Agent

Applied the change from an **Administrator** PowerShell session:

```powershell
NET STOP wazuhSvc; NET START wazuhSvc
```

### 3. Simulate Security Events

Ran PowerShell commands to create, modify, and delete a file:

```powershell
# Create initial file
"Initial text" | Out-File -FilePath "C:\wazuh_test\test.txt"
Start-Sleep -Seconds 3

# Modify file content
"Modified text string" | Out-File -FilePath "C:\wazuh_test\test.txt" -Append
Start-Sleep -Seconds 3

# Delete file
Remove-Item -Path "C:\wazuh_test\test.txt"
```

---

## 📊 Verification & Results

Each file operation triggered a real-time **syscheck** event in the Wazuh Dashboard:

| Action | Syscheck Event | Rule Description |
|--------|----------------|------------------|
| File creation | `added` | File added to the system. |
| Modification | `modified` | Integrity checksum changed. |
| Deletion | `deleted` | File deleted. |

<!-- Add screenshots here, e.g.:
![Added event](screenshots/added.png)
![Modified event](screenshots/modified.png)
![Deleted event](screenshots/deleted.png)
-->

---

## 🔒 Security Takeaways

- **Proactive tampering detection:** Real-time FIM alerts analysts immediately when unauthorized files are added or modified in monitored directories.
- **Compliance & auditing:** FIM supports requirements such as **PCI-DSS** and **SOC 2** by maintaining an audit trail of file system integrity.

---

## 🧰 Tools Used

`Wazuh` · `Ubuntu Linux` · `Windows` · `PowerShell` · 
