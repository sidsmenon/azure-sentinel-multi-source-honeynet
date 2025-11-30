# From Honeypot to Honeynet: Multi-Source Azure Sentinel Lab (Windows + Linux + Attacker VM)

A hands-on Azure Sentinel SIEM lab that evolves a basic honeypot into a small **honeynet**. This environment generates **repeatable attack telemetry on demand** (RDP/SSH/MSSQL brute-force style failures) while also ingesting **Azure control-plane and data-plane logs** (Entra ID/Azure AD, Storage, Key Vault, Activity Logs) into a single Log Analytics workspace. The result is a set of **four Sentinel workbooks** that provide a clean, multi-signal view of security activity across hosts, network exposure, and Azure services.

> This project is a **v2 / expanded edition** of my earlier Azure Sentinel honeypot project (failed RDP + world map).  
> If you want the simpler baseline first: **[Old Project: Azure Sentinel Honeypot Threat Analysis](https://github.com/sidsmenon/sentinel-honeypot)**


<img width="906" height="635" alt="image" src="https://github.com/user-attachments/assets/c0871a2c-bbdc-4e5b-b155-baa5568bbb82" />


## Architecture (High Level)

**Attack generation**
- **Attacker VM (Windows)** simulates:
  - Failed RDP attempts against Windows victim
  - Failed MSSQL login attempts against MSSQL on Windows victim

**Victims / telemetry sources**
- **Windows Victim VM**
  - RDP auth failures
  - MSSQL auth failures (SQL Server instance installed)
- **Linux Victim VM**
  - Failed SSH attempts

**Azure platform signals**
- **Entra ID / Azure AD** – audit logs (user CRUD, etc.)
- **Azure Storage (Blob)** – activity/diagnostic logs (CRUD on storage, data access as configured)
- **Azure Key Vault** – diagnostic logs (secrets/keys operations)
- **Azure Activity Logs** – subscription-level operations (e.g., RG CRUD)
- **Network flow visibility**
  - **VNet Flow Logs + Traffic Analytics** → `NTANetAnalytics` table (modern replacement for NSG flow logs)

All logs are centralized in **Log Analytics Workspace → Microsoft Sentinel → Workbooks / Hunting / Analytics**.

---

## What You’ll Build

### ✅ Four Sentinel Workbooks
1. **Failed RDP Logins (Windows)**  
2. **Vulnerable Network Exposure / Flow Logs (VNet Flow Logs / Traffic Analytics)**  
3. **Failed SSH Attempts (Linux)**  
4. **Failed MSSQL Login Attempts (Windows SQL Server)**  

Each workbook is designed to be focused, readable, and map-friendly (where applicable).

---

## Why This Lab?

My first Sentinel lab stopped at a single data source and a map. This project is meant to feel closer to a real SOC workflow:
- multiple host signals (Windows + Linux)
- repeatable attack generation (not just waiting for the internet)
- identity + secrets + storage + subscription activity signals in one workspace
- workbook-driven visibility for quick triage

---

## Data Sources & Tables (What to Expect)

Depending on how you onboard each source, you’ll typically see data land in tables like:
- **Windows Security Events**: `SecurityEvent` *(Sentinel connector via AMA)*  
- **Linux Syslog / Auth**: `Syslog`  
- **Azure Activity Logs**: `AzureActivity`  
- **Entra ID / Azure AD logs**: `SigninLogs`, `AuditLogs` *(if enabled/connected)*  
- **Key Vault diagnostics**: `AzureDiagnostics` (ResourceType = KeyVault) or resource-specific tables
- **Storage diagnostics**: `AzureDiagnostics` or storage-specific tables
- **VNet Flow Logs (Traffic Analytics)**: `NTANetAnalytics`

> Note: Azure logging options and table names can vary by connector mode, diagnostic setting type, and region.

---

## Setup Guide (Step-by-Step)



## Security Notice / Disclaimer

This lab intentionally reduces security controls to attract or simulate attacks (open ports, permissive NSGs, etc.).  
**Do not deploy this in a production subscription** or any environment with sensitive data.

Use a dedicated lab subscription, strict budget alerts, and tear down resources after testing.


## Roadmap / Improvements
- Add detections (Analytics rules) for:
  - repeated auth failures (brute-force)
  - risky Key Vault access
  - unusual Storage operations
  - suspicious subscription-level changes
- Add automation (Logic Apps) for alert triage
- Add entity mapping and incident workflows
- Add MITRE ATT&CK mapping
