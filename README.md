# Centralized SOC & SIEM Lab Environment

A hands-on, multi-node cybersecurity lab integrating Windows 11 log telemetry, Kali Linux security tools, an Ubuntu-hosted Splunk SIEM, and RustDesk unattended remote access for team collaboration.

---

## 📐 Architecture Overview

```text
+-------------------------------------------------------+
|                   HOST ENVIRONMENT                    |
|                                                       |
|  +--------------------+        +-------------------+  |
|  | Windows 11 Host    |        | Kali Linux VM     |  |
|  | (192.168.43.130)   |        | (192.168.43.130)  |  |
|  | - Sysmon / UF      |        | - Penetration     |  |
|  +---------+----------+        |   Testing Tools   |  |
|            |                   | - RustDesk Host   |  |
|            | TCP 9997          +---------+---------+  |
|            |                             |            |
|            v                             |            |
|  +--------------------+                  | Remote     |
|  | Ubuntu SIEM        |                  | Access     |
|  | (192.168.43.128)   |                  |            |
|  | - Splunk Core      |                  v            |
|  +--------------------+         +------------------+  |
|                                 | External         |  |
|                                 | Colleagues       |  |
|                                 | (RustDesk Client)|  |
|                                 +------------------+  |
+-------------------------------------------------------+
```

---

## 🛠️ Components & IP Scheme

| Node Name | Operating System | IP Address | Function / Role |
| :--- | :--- | :--- | :--- |
| **SIEM Server** | Ubuntu Server/Desktop | `192.168.43.128` | Centralized Splunk Indexer & Web Console |
| **Endpoint Host** | Windows 11 | `192.168.43.130` | Target workstation running Sysmon & Universal Forwarder |
| **Attack Node** | Kali Linux VM | `192.168.43.130` | Security testing node configured with RustDesk for remote access |

---

## 🚀 Implementation Steps

### Phase 1: Infrastructure & Network Architecture
1. **Network Topology:** Established a local subnetwork allowing inter-VM routing while isolating log traffic.
2. **IP Addressing:** Configured static IP mappings across all nodes for reliable endpoint identification.
3. **Connectivity Verification:** Validated unidirectional TCP connectivity on **port 9997** from the Windows endpoint to the Ubuntu Splunk indexer.

---

### Phase 2: Log Forwarding & Ingestion (Splunk SIEM)
1. **Service Configuration:** Installed the Splunk Universal Forwarder on Windows 11 and assigned it to run under the **Local System Account** to allow proper reading privileges for restricted Windows Event Logs (Sysmon/Security).
2. **Outputs Setup (`outputs.conf`):** Configured the target indexer destination:
   ```ini
   [tcpout]
   defaultGroup = default-autolb-group

   [tcpout:default-autolb-group]
   server = 192.168.43.128:9997
   ```
3. **Inputs Setup (`inputs.conf`):** Enabled Sysmon operational log collection with XML rendering:
   ```ini
   [default]
   host = windows11-host

   [WinEventLog://Microsoft-Windows-Sysmon/Operational]
   disabled = 0
   renderXml = true
   index = default
   ```
4. **Cache & Queue Clearing:** Purged checkpoint persistent queues (`persistentqueue`) on the forwarder to guarantee real-time ingestion upon service restart.

---

### Phase 3: Unattended Remote Access & Collaboration (RustDesk)
1. **Network Adapter Setup:** Configured Kali Linux to NAT mode to ensure unrestricted internet connectivity for package installations.
2. **RustDesk Installation:** Installed RustDesk on the Kali VM via Debian package management:
   ```bash
   wget https://github.com/rustdesk/rustdesk/releases/download/1.3.0/rustdesk-1.3.0-x86_64.deb
   sudo apt update && sudo apt install ./rustdesk-1.3.0-x86_64.deb
   ```
3. **Unattended Access:** Configured RustDesk with a **Permanent Password** and adjusted Linux display server settings to allow colleagues to seamlessly connect from external networks at any time without manual on-screen approval.

---

## 🔍 Log Ingestion Verification

To confirm telemetry streaming from Windows 11 to the Ubuntu Splunk SIEM:

```text
index=* sourcetype="*Sysmon*"
```

---

## 👤 Author & Team
* **Lab Architect:** Ahmmad Alkufahy, Mohammad AlNabelsi, Ahmad Migdadi
* **Collaboration Tooling:** RustDesk for persistent cross-network remote access.
