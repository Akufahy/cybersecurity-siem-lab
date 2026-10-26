# Centralized SOC & SIEM Lab Environment & Incident Readiness Pack
A hands-on, multi-node cybersecurity lab integrating Windows 11 log telemetry, Kali Linux system auditing (`auditd` & Nginx web service logs forwarded via Splunk Universal Forwarder), an Ubuntu-hosted Splunk SIEM, and RustDesk unattended remote access for team collaboration, paired with an organizational Incident Readiness Pack for **Al Amal Hospital**.

## 📐 Architecture Overview
```text
+-------------------------------------------------------+
|                   HOST ENVIRONMENT                    |
|                                                       |
|  +--------------------+        +-------------------+  |
|  | Windows 11 Host    |        | Kali Linux VM     |  |
|  | (192.168.43.131)   |        | (192.168.43.129)  |  |
|  | - Sysmon / UF      |        | - auditd & Nginx  |  |
|  +---------+----------+        | - Splunk Forwarder|  |
|            |                   | - RustDesk Host   |  |
|            | TCP 9997          +---------+---------+  |
|            |                             | TCP 9997   |
|            +--------------+--------------+            |
|                           |                           |
|                           v                           |
|                +--------------------+                 |
|                | Ubuntu SIEM        |                 |
|                | (192.168.43.128)   |                 |
|                | - Splunk Core      |                 |
|                +--------------------+                 |
+-------------------------------------------------------+
```

## 🛠️ Components & IP Scheme
| Node Name | Operating System | IP Address | Function / Role |
| :--- | :--- | :--- | :--- |
| **SIEM Server** | Ubuntu Server/Desktop | 192.168.43.128 | Centralized Splunk Indexer & Web Console |
| **Endpoint Host** | Windows 11 | 192.168.43.131 | Target workstation running Sysmon & Universal Forwarder |
| **Service Node** | Kali Linux VM | 192.168.43.129 | Web service (Nginx), system auditing (`auditd`), and Splunk Universal Forwarder |

## 🚀 Implementation Steps
### Phase 1: Infrastructure & Network Architecture
1. **Network Topology:** Established a local subnetwork allowing inter-VM routing while isolating log traffic.
2. **IP Addressing:** Configured static IP mappings across all nodes for reliable endpoint identification.
3. **Connectivity Verification:** Validated unidirectional TCP connectivity on **port 9997** from both Windows 11 and Kali endpoints to the Ubuntu Splunk indexer.

### Phase 2: Log Forwarding & Ingestion (Splunk SIEM)
1. **Windows Service Configuration:** Installed the Splunk Universal Forwarder on Windows 11 under the **Local System Account** to ingest restricted Windows Event Logs (Sysmon/Security).
2. **Linux Service Configuration:** Installed Splunk Universal Forwarder on Kali Linux to monitor and stream `/var/log/audit/audit.log` (`sourcetype=linux_audit`) and Nginx logs directly to Splunk Core.
3. **Outputs Setup (`outputs.conf`):** Configured the target indexer destination across forwarders:
   ```ini
   [tcpout]
   defaultGroup = default-autolb-group

   [tcpout:default-autolb-group]
   server = 192.168.43.128:9997
   ```
4. **Inputs Setup (`inputs.conf`):** Enabled log collection on forwarders:
   ```ini
   # Windows 11 Forwarder
   [WinEventLog://Microsoft-Windows-Sysmon/Operational]
   disabled = 0
   renderXml = true
   index = default

   # Kali Linux Forwarder
   [monitor:///var/log/audit/audit.log]
   disabled = 0
   sourcetype = linux_audit
   index = default
   ```

### Phase 3: Unattended Remote Access & Collaboration (RustDesk)
1. **Network Adapter Setup:** Configured Kali Linux to NAT mode to ensure unrestricted internet connectivity for package installations.
2. **RustDesk Installation:** Installed RustDesk on the Kali VM via Debian package management:
   ```bash
   wget https://github.com/rustdesk/rustdesk/releases/download/1.3.0/rustdesk-1.3.0-x86_64.deb
   sudo apt update && sudo apt install ./rustdesk-1.3.0-x86_64.deb
   ```
3. **Unattended Access:** Configured RustDesk with a **Permanent Password** and adjusted Linux display server settings to allow colleagues to seamlessly connect from external networks at any time.

## 🛡️ Part 2: Incident Readiness Pack — Al Amal Hospital
The repository includes the complete Incident Readiness Pack tailored for **Al Amal Hospital** (300-bed regional acute care and trauma center):
* **Task 2.1 (`Task_2.1_Organisation_Profile_and_Risk_Register.docx`):** Hospital profile, system MTDs, and 10-risk register mapped to Part 1 rules.
* **Task 2.2 (`Task_2.2_Severity_Matrix_and_Card_Updates.docx`):** P1–P4 Severity Matrix with 12 healthcare scenarios and Part 1 card updates.
* **Task 2.3 (`Task_2.3_Escalation_Tree_and_External_Reporting.docx`):** Internal escalation pathways, out-of-hours on-call rotation, and national/regulatory reporting (NCSC / JO-CERT).
* **Task 2.4 (`Task_2.4_Incident_Response_Playbooks.docx`):** 8-stage operational playbooks for Top 3 Risks (Encoded PowerShell R02, Firewall Modification R05, Log Tampering R06).
* **Task 2.5 (`Task_2.5_Playbook_Testing_and_Cold_Test.docx`):** Timed internal dry-runs and peer student cold-test evaluation.
* **Task 2.6 (`Task_2.6_Communication_Plan_and_Templates.docx`):** Stakeholder communication governance and pre-approved P1 message templates.
* **Task 2.7 (`Task_2.7_Tabletop_Exercise_and_Injects.docx`):** 45-minute Tabletop Exercise (TTX) featuring 6 chronological injects (including ransomware and media pressure).
* **Task 2.8 (`Task_2.8_Executive_Briefing_and_Business_Case.docx`):** Executive briefing summary and financial/operational business case.

## 🔍 Log Ingestion Verification
To confirm telemetry streaming from both Windows 11 and Kali Linux forwarders to the Ubuntu Splunk SIEM:
```text
index=* sourcetype="*Sysmon*"
index=* sourcetype="linux_audit"
```

## 👤 Author & Team
- Lab Architect & Project Team: Ahmmad Alkufahy, Mohammad AlNabelsi, Ahmad Migdadi
- Collaboration Tooling: RustDesk for persistent cross-network remote access.
