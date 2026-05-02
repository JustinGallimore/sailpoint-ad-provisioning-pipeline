# SailPoint ISC — Active Directory Provisioning Pipeline
### Enterprise IAM Lab | Phase 2 of 3

![SailPoint](https://img.shields.io/badge/SailPoint-ISC-blue)
![Active Directory](https://img.shields.io/badge/Active%20Directory-Windows%20Server%202022-0078D4)
![IQService](https://img.shields.io/badge/IQService-Build%20842-green)
![VMware](https://img.shields.io/badge/VMware-Workstation%2016%20Pro-607078)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)

---

## What This Project Demonstrates

This project builds a production-grade Active Directory provisioning pipeline using SailPoint IdentityNow (ISC). It extends the HR-driven JML automation built in Phase 1 by connecting SailPoint to a live Windows Server 2022 domain controller and configuring automated AD account creation through IQService.

The lab covers the full integration stack from infrastructure build to identity lifecycle automation, including real-world troubleshooting of VM networking, IQService TLS configuration, and certificate chain management. Every step reflects the kind of work IAM engineers perform in enterprise environments every day.

---

## Architecture Overview

```
CSV HR Source (SFTP)
        |
        v
SailPoint ISC Tenant (Cloud)
        |
        v
Virtual Appliance — VMware (192.168.50.211)
        |
        v
IQService — Windows Server 2022 DC01 (192.168.50.10) — TLS Port 5050
        |
        v
Active Directory — lab.local
        |
        v
OU=LabUsers > Engineering / Finance / HR / IT / Marketing
```

---

## Lab Network Map

| Host | IP Address | Role |
|------|-----------|------|
| Xfinity Router | 192.168.50.1 | Default Gateway |
| DC01 | 192.168.50.10 | Windows Server 2022 Domain Controller |
| POWERSTATION-7 | 192.168.50.91 | Windows 11 Host Machine |
| SailPoint VA | 192.168.50.211 | Virtual Appliance (VMware) |

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| SailPoint ISC | Identity governance and provisioning platform |
| Windows Server 2022 | Domain controller hosting Active Directory |
| Active Directory | Target system for automated account provisioning |
| IQService Build 842 | SailPoint provisioning agent on DC01 |
| VMware Workstation 16 Pro | Hypervisor hosting DC01 and SailPoint VA |
| PowerShell | Infrastructure configuration and troubleshooting |
| SailPoint Virtual Appliance | Connector gateway between ISC and on-prem AD |

---

## What Was Built

### Infrastructure
- Deployed Windows Server 2022 in VMware Workstation with 4GB RAM, 4 cores, 60GB NVMe, bridged networking
- Installed Active Directory Domain Services and promoted DC01 to domain controller
- Created new forest and domain: lab.local with Windows Server 2016 functional level
- Built OU structure under LabUsers with sub-OUs for Engineering, Finance, HR, IT, and Marketing
- Configured static IP at 192.168.50.10 after resolving VMware bridged adapter issues

### Virtual Appliance Migration
- Original SailPoint VA was running in VirtualBox and could not reach DC01 in VMware despite both using bridged networking on the same physical adapter
- Diagnosed the issue via SSH and confirmed 100% packet loss from VA to DC01
- Migrated the VA from VirtualBox to VMware by converting VDI to VMDK using VBoxManage
- After migration the VA picked up IP 192.168.50.211 and confirmed 0% packet loss to DC01

### SailPoint AD Source Configuration
- Created AD_Lab_DC01 source with Forest Settings pointing to 192.168.50.10 on port 3268
- Configured Domain Settings with DC=lab,DC=local on port 389
- Built account schema with 88 attributes and configured account correlation using Work Email equals mail rule
- Ran first aggregation and discovered 4 AD accounts
- Created AD_LabUsers_Access access profile with Domain Users entitlement
- Attached access profile to the Active lifecycle state in HR_CSV_Lab_Identity_Profile

### IQService Installation and TLS Configuration
- Downloaded IQService Build 842 from SailPoint ISC admin panel
- Installed IQService on DC01 which registered two Windows services on TLS ports 5050 and 5051
- Generated a self-signed TLS certificate using New-SelfSignedCertificate with DnsName DC01.lab.local
- Bound the certificate to port 5050 using netsh with the correct thumbprint and application GUID
- Configured IQService Settings in ISC with host 192.168.50.10, port 5050, TLS enabled

### JML Workflow Execution
- Added Emily Johnson as employee JG9006 to the CSV HR source with department HR and status active
- Ran account aggregation on HR_Source_CSV_Lab
- Emily's identity was created in SailPoint with Active lifecycle state and all 8 attributes populated correctly
- Joiner workflow fired automatically and attempted AD account creation
- Provisioning reached IQService and attempted to create the account in the LabUsers OU

### Provisioning Policy Configuration
- Configured Create Account provisioning policy with distinguishedName pointing to OU=LabUsers,DC=lab,DC=local
- Mapped displayName to Display Name and mail to Work Email
- Updated sAMAccountName from the Create Unique LDAP Attribute generator to a Static value of ejohnson to eliminate LDAP uniqueness check timeouts

---

## Current Status and Known Limitation

The provisioning pipeline is fully configured and operational through the IQService layer. The Joiner workflow fires correctly, the identity is created with the correct attributes, and SailPoint successfully reaches IQService on DC01.

The AD account creation is currently blocked by a TLS certificate trust issue. IQService Build 842 requires TLS and installs with TLS enforced on port 5050. The self-signed certificate generated on DC01 is correctly bound to the port, however the SailPoint Virtual Appliance does not trust self-signed certificates by default. The VA is a locked-down appliance that restricts the sailpoint user from importing certificates into the OS trust store or the Java truststore without root access.

In a production environment this is resolved by issuing the IQService certificate from an internal certificate authority that the VA already trusts, eliminating the need to manually import anything. This is standard practice in enterprise deployments and is the documented SailPoint recommendation for IQService TLS configuration.

---

## Key IAM Concepts Demonstrated

- Virtual appliance deployment and network troubleshooting in a hybrid environment
- Active Directory source integration with SailPoint ISC including schema and correlation configuration
- IQService architecture, installation, and TLS certificate management
- Provisioning policy design including attribute mapping and account naming conventions
- End-to-end Joiner workflow execution with real-time event logging
- Enterprise-level troubleshooting methodology including log analysis, certificate chain tracing, and network diagnostics

---

## Repository Structure

```
sailpoint-ad-provisioning-pipeline/
├── README.md
├── Walkthrough.md
└── screenshots/
    ├── 01_vmware_new_vm_wizard.png
    ├── 02_vmware_custom_hardware_wizard.png
    └── ... (86 screenshots total)
```

---

## Related Projects

- Phase 1: [SailPoint JML Pipeline](https://github.com/JustinGallimore/sailpoint-jml-pipeline) — HR-driven Joiner Mover Leaver automation using CSV source over SFTP
- Phase 3: SailPoint Governance Program — RBAC design, SoD policies, access certifications, and SOX audit evidence (Coming Soon)

---

## Author

Justin Gallimore
IAM Engineer | SailPoint ISC | Active Directory | Identity Lifecycle Automation
[LinkedIn](https://linkedin.com/in/justingallimore) | [GitHub](https://github.com/JustinGallimore)
