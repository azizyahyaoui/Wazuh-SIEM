# 🛡️ Wazuh SIEM & Security Operations Course

[![Wazuh](https://img.shields.io/badge/Wazuh-v4.x-blue.svg?style=for-the-badge&logo=wazuh)](https://wazuh.com/)
[![SIEM](https://img.shields.io/badge/SIEM-Security%20Operations-red.svg?style=for-the-badge&logo=shield)](https://wazuh.com/)
[![EDR](https://img.shields.io/badge/EDR-Endpoint%20Detection-orange.svg?style=for-the-badge)](https://wazuh.com/)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK%20Mapped-success.svg?style=for-the-badge)](https://attack.mitre.org/)
[![License](https://img.shields.io/badge/License-MIT-brightgreen.svg?style=for-the-badge)](LICENSE)

Welcome to the **Wazuh SIEM & Security Operations Course** repository. This project provides a comprehensive learning path, hands-on lab architecture, and detailed documentation on implementing and managing **Wazuh** — an open-source Unified XDR (Extended Detection and Response) and SIEM platform.

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Course Curriculum](#-course-curriculum)
- [Key Features & Capabilities](#-key-features--capabilities)
- [Wazuh Architecture](#-wazuh-architecture)
- [Repository Structure](#-repository-structure)
- [Quick Start](#-quick-start)
- [Contributing & License](#-contributing--license)

---

## 🧐 Overview

Modern Security Operations Centers (SOC) require full visibility across endpoints, cloud workloads, network traffic, and containerized environments. This course bridges the gap between theoretical SIEM concepts and hands-on detection engineering using **Wazuh**.

---

## 📚 Course Curriculum

The main course material is available in [`course/WazuhSIEM.md`](./course/WazuhSIEM.md) and covers:

1. **Part 1: Introduction to SIEM**
   - What is SIEM? (SIM vs. SEM)
   - Core SIEM Functions: Log Aggregation, Parsing/Normalization, Real-Time Event Correlation, Threat Intel Enrichening, and Compliance Reporting.
2. **Part 2: Introduction to Wazuh**
   - Core Architecture: Agent, Manager/Server, Indexer (OpenSearch), and Dashboard.
   - Endpoint Detection & Response (EDR) and File Integrity Monitoring (FIM).
   - Vulnerability Detection & MITRE ATT&CK framework mapping.
   - Active Response & Automated Incident Mitigation.
   - Traditional SIEM vs. Wazuh Unified XDR comparison.

---

## ⚡ Key Features & Capabilities

| Feature Category | Capabilities |
| :--- | :--- |
| **Endpoint Security (EDR/XDR)** | File Integrity Monitoring (FIM), rootkit detection, process auditing, user activity tracking. |
| **Log Data Analysis** | Multi-source log collection, normalization, custom decoders, and correlation rule sets. |
| **Vulnerability Detection** | Automated CVE scanning for OS and third-party software against National Vulnerability Databases. |
| **MITRE ATT&CK Mapping** | Native mapping of all alerts to Tactics, Techniques, and Procedures (TTPs). |
| **Active Response** | Triggering local firewall blocks, process termination, or endpoint isolation upon threat detection. |
| **Compliance Auditing** | Out-of-the-box dashboards for **PCI-DSS**, **GDPR**, **SOC 2**, **HIPAA**, **NIST 800-53**, and **CIS Benchmarks**. |

---

## 🏗️ Wazuh Architecture

```mermaid
graph LR
    subgraph "Monitored Endpoints"
        E1["Windows Endpoints"]
        E2["Linux Servers"]
        E3["macOS / Unix"]
        E4["Cloud & Syslog Devices"]
    end

    subgraph "Wazuh Platform"
        W_Agent["Wazuh Agent"]
        W_Server["Wazuh Manager / Server"]
        W_Indexer["Wazuh Indexer (OpenSearch)"]
        W_Dashboard["Wazuh Web Dashboard"]
    end

    E1 --> W_Agent
    E2 --> W_Agent
    E3 --> W_Agent
    E4 -- "Syslog / APIs" --> W_Server
    W_Agent -- "Port 1514 (Encrypted)" --> W_Server
    W_Server -- "Decoded Alerts" --> W_Indexer
    W_Dashboard -- "Search & Viz" --> W_Indexer
    W_Dashboard -- "API Management" --> W_Server
```

---

## 📁 Repository Structure

```text
Wazuh SIEM/
├── course/
│   ├── WazuhSIEM.md          # Complete SIEM & Wazuh documentation and notes
│   ├── Images/               # Course diagrams and graphics
│   ├── pdf/                  # Exported PDF documentation
│   └── screenshots/          # Hands-on lab screenshots
├── docker/                   # Docker Compose environment (coming soon)
└── README.md                 # Project Overview & Guide
```

---

## 🚀 Quick Start

1. **Read the Course Material**:
   Open [`course/WazuhSIEM.md`](./course/WazuhSIEM.md) to explore the SIEM fundamentals and Wazuh architecture breakdown.

2. **Deploying Wazuh (Docker)** *(Optional)*:
   Check the `docker/` folder for upcoming Docker Compose deployment scripts to launch a single-node or multi-node Wazuh stack locally.

---

## 📄 License

This repository is maintained for educational purposes. Feel free to use and adapt the material.
