# enterprise-IT-infrastructure-design
Hyper‑V lab simulating an SME network using a NAT‑isolated 10.0.0.0/24. Two Windows Server 2019 domain controllers provide AD, DNS, and DHCP with redundancy. A separate file server uses group‑based access and GPO drive mapping, managed via an RSAT workstation.


Enterprise Active Directory Home Lab

Architecture - Build - Operations

Overview:-

This repository documents an enterprise‑style Active Directory (AD) and Windows infrastructure home lab built using Microsoft Hyper‑V.
The lab is intentionally designed to replicate real corporate environments, focusing on identity, access management, security boundaries, and operational best practices, rather than a minimal or academic setup.

The goal of this lab is to demonstrate Operational Design around:

Active Directory design
Least‑privilege administration
Group‑based access control
Group Policy–driven configuration
Real‑world troubleshooting and validation


Lab Objectives

Build a redundant Active Directory environment
Separate critical roles (identity, file services, administration)
Implement group‑based NTFS permissions
Deliver user experience via Group Policy
Simulate Joiner / Mover / Leaver (JML) workflows
Practice troubleshooting scenarios commonly seen in enterprise IT


High‑Level Architecture
Core components:

Hyper‑V host with internal virtual switch + NAT
Two Windows Server domain controllers (DC01, DC02)
Dedicated file server (FS01)
Dedicated administrative workstation (RSAT)
Domain‑joined user workstations


Key principle:
Administration is performed from a secured workstation — not directly on domain controllers.


Virtualization & Networking
Hyper‑V Configuration

Virtual switch type: Internal
Host‑based NAT used for controlled internet access

IP Addressing
Network: 10.0.0.0/24
Gateway (Host NAT): 10.0.0.1

This design isolates lab traffic while ensuring domain members can:

Contact domain controllers
Access the internet for updates and testing


Server Inventory
DC01 – Primary Domain Controller

OS: Windows Server 2019
IP: 10.0.0.10
Roles:

Active Directory Domain Services
DNS (AD‑integrated)
DHCP


Purpose:

Forest and domain creation
Primary authentication authority




DC02 – Additional Domain Controller

OS: Windows Server 2019
IP: 10.0.0.11
Roles:

AD DS
DNS


Purpose:

Redundancy
Replication validation
Authentication resilience



Replication health validated using:
PowerShellrepadmin /replsummarydcdiag``Show more lines

FS01 – File Server (Member Server)

OS: Windows Server 2019
IP: 10.0.0.20
Role: File Services

Storage Layout
C:  Operating system
D:  Data volume
    └── D:\Shares
        ├── IT
        └── HR


Design decision:
Data is isolated from the OS volume, mirroring enterprise storage practices.


Administrative Workstation

OS: Windows 11 Pro
Tools installed:

RSAT
Active Directory Users and Computers
Group Policy Management Console


Purpose:

Centralised and secure administration
Reduced attack surface on DCs




User Workstations

Windows 10 / Windows 11
Domain‑joined
Used to validate:

Logon behavior
Group Policy application
Drive mapping
File access




Active Directory Design
Domain
Domain name: contoso..com

OU Structure
Logical OU separation was implemented for:

Users (by department)
Computers
Servers
Administrative accounts

This enables precise GPO targeting and avoids policy sprawl.

Security Group Model
All access is granted via security groups, never directly to users.
Department Groups

SG-IT-Users
SG-IT-Managers
SG-HR-Users
SG-HR-Managers

Global Group

SG-Contoso-AllEmployees


Rule enforced:
Access requests are resolved by group membership changes only.


File Services & NTFS Permissions
Root Share (D:\Shares)

Read & Execute permission for SG-Contoso-AllEmployees
Applies to This folder only
Allows traversal without data exposure

Department Folder Model (Example: HR)


Principal          Permission                Scope 
SYSTEM              Full              Folder, Subfolders, Files 
Administrators      Full              Folder, Subfolders, Files 
SG-HR-Managers      Full              Folder, Subfolders, Files 
SG-HR-Users         Modify            Folder, Subfolders, Files 
CREATOR OWNER       Modify            Subfolders and Files only

This ensures:

Users can create, edit, rename, and delete files
Permissions remain scalable and auditable
No per‑user NTFS changes are required


Group Policy Design
Domain Baseline Policy

Password policy
Account lockout policy
Enforced at domain level

Admin Workstation Security Policy

Applied only to admin workstation OU
Blocks standard user logons
Reinforces least‑privilege administration

Drive Mapping via Group Policy Preferences

User Configuration → Drive Maps
Action: Replace (prevents stale mappings)
Item‑level targeting based on department groups

Example:
H: → \\FS01\HR$
I: → \\FS01\IT$


Operational Workflows (JML)
Joiner

Create user in correct OU
Add to department group(s)
Drive mappings and access apply automatically

Mover

Remove old department group
Add new department group
Access changes without touching file server or NTFS

Leaver

Disable account
Remove group memberships
Access revoked everywhere instantly


DNS, DHCP & Authentication Flow
DNS

AD‑integrated on DC01 and DC02
Secure dynamic updates
Forwarders configured for external resolution

DHCP

Hosted on DC01
Provides:

Gateway: 10.0.0.1
DNS: 10.0.0.10, 10.0.0.11
DNS suffix: contoso.com



Clients never use public DNS directly.

Troubleshooting & Validation
Real issues were encountered and resolved, including:

Drive mapping not applying due to GPO action type
NTFS inheritance causing edit/rename failures
Domain logon issues caused by startup timing
Admin workstation lockout due to mis‑scoped policies

Validation tools used:
PowerShelldcdiagrepadmin /replsummarygpupdate /forcegpresult /rShow more lines

Final State
This lab represents a complete, enterprise‑style Windows infrastructure environment with:

Redundant AD design
Secure delegated administration
Group‑based file access
Policy‑driven user experience
Realistic operational workflows and troubleshooting
