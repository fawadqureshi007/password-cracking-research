# Attack Guide

> Practical reference for password-security research, authentication auditing, credential-security testing, and authorized security assessments.

---

## Overview

This directory contains practical guides for tools used in:

```text
Password Security Research
Authentication Testing
Credential Auditing
Hash Recovery
Windows Security Assessment
Active Directory Security Testing
Network Authentication Testing
```

All techniques should be performed only in environments where you have explicit authorization.

---

# Tool Collection

## Password & Hash Analysis

### Hashcat

[`Hashcat.md`](Hashcat.md)

GPU-accelerated password-recovery and hash-analysis tool.

Topics include:

```text
Hash identification
Hash modes
Wordlists
Mask-based candidates
Rule-based processing
Benchmarking
Recovery workflows
```

---

### John the Ripper

[`john-the-ripper.md`](john-the-ripper.md)

Password-security auditing and offline hash-recovery framework.

Topics include:

```text
Hash formats
Wordlists
Rules
Incremental modes
Custom candidates
Session management
Result analysis
```

---

### Hashcat Utils

[`hashcat-utils.md`](hashcat-utils.md)

Utilities for preparing and processing password candidates for authorized research.

Topics include:

```text
Candidate generation
Wordlist processing
Charset preparation
Password-data transformation
Hashcat workflow support
```

---

### Ophcrack

[`ophcrack.md`](ophcrack.md)

Windows password-recovery research tool based on rainbow-table techniques.

Topics include:

```text
Windows password auditing
Rainbow tables
NTLM research
Laboratory recovery
Result interpretation
```

---

### RainbowCrack

[`rainbowcrack.md`](rainbowcrack.md)

Rainbow-table based password-recovery research.

Topics include:

```text
Rainbow tables
Hash generation
Table creation
Table lookup
Password-recovery research
```

---

# Authentication Testing

## Hydra

[`hydra.md`](hydra.md)

Parallelized network authentication-security testing tool.

Common laboratory protocols include:

```text
SSH
FTP
Telnet
SMTP
IMAP
POP3
MySQL
PostgreSQL
SMB
RDP
HTTP
HTTPS
```

The guide focuses on:

```text
Authentication testing
Module usage
Laboratory services
Rate-limit research
Account-lockout testing
Authentication logging
Detection research
```

---

## Medusa

[`medusa.md`](medusa.md)

Parallel network-login auditing tool for authorized authentication testing.

Topics include:

```text
Module usage
Credential auditing
Network services
Laboratory authentication
Logging
Rate limiting
```

---

## Ncrack

[`ncrack.md`](ncrack.md)

Network authentication auditing tool designed for controlled security assessments.

Topics include:

```text
Supported services
Authentication testing
Service configuration
Laboratory testing
Credential-security assessment
```

---

## Patator

[`patator.md`](patator.md)

Modular brute-force and authentication-testing framework.

Topics include:

```text
Module architecture
Authentication testing
Parameter handling
Laboratory workflows
Request analysis
Result interpretation
```

---

## Crowbar

[`crowbar.md`](crowbar.md)

Credential-testing tool focused on supported remote authentication services.

Topics include:

```text
RDP testing
SSH-key testing
OpenVPN testing
Credential auditing
Laboratory environments
```

---

# Windows & Active Directory Security

## CrackMapExec

[`crackmapexec.md`](crackmapexec.md)

Windows and Active Directory security-assessment framework.

Topics include:

```text
SMB enumeration
Windows host identification
SMB signing
Share auditing
WinRM
LDAP
MSSQL
RDP
Credential validation
Privilege review
Active Directory assessment
Windows logging
Detection engineering
```

> CrackMapExec is primarily a Windows/Active Directory security-assessment tool rather than a standalone password-cracking utility.

---

# Recommended Learning Path

Start with offline password research:

```text
Hashcat
   ↓
John the Ripper
   ↓
Hashcat Utils
   ↓
Ophcrack
   ↓
RainbowCrack
```

Then move into authentication testing:

```text
Hydra
   ↓
Medusa
   ↓
Ncrack
   ↓
Patator
   ↓
Crowbar
```

Then Windows and Active Directory:

```text
CrackMapExec
   ↓
SMB Security
   ↓
WinRM
   ↓
LDAP
   ↓
Active Directory Assessment
```

---

# Laboratory Structure

A safe research environment can contain:

```text
Attack-Testing-VM
        │
        ├── Kali / Parrot OS
        │
        └── Testing Tools
                 │
                 ▼
        ┌──────────────────┐
        │ Isolated Network │
        └──────────────────┘
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
      Linux    Windows   AD Lab
       VM        VM        DC
```

Keep vulnerable systems and test accounts inside an isolated environment.

---

# Research Workflow

Use the following methodology:

```text
1. Define Scope
       ↓
2. Obtain Authorization
       ↓
3. Build Laboratory
       ↓
4. Create Test Accounts
       ↓
5. Identify Target Service
       ↓
6. Select Appropriate Tool
       ↓
7. Perform Controlled Test
       ↓
8. Monitor Logs
       ↓
9. Record Results
       ↓
10. Identify Security Weaknesses
       ↓
11. Apply Remediation
       ↓
12. Retest
       ↓
13. Document Findings
```

---

# What to Record

For repeatable security research, record:

```text
Tool
Version
Operating System
Target
Protocol
Test Account
Candidate Dataset
Configuration
Start Time
End Time
Result
Server Logs
Detection Result
Security Finding
Remediation
```

Never store real production credentials in research notes.

---

# Defensive Research

These tools can also be used to evaluate defensive controls.

Research areas include:

```text
Password Policy
Password Reuse
Account Lockout
Rate Limiting
MFA
SMB Signing
LDAP Security
WinRM Restrictions
RDP Security
Authentication Logging
SIEM Detection
Alerting
Least Privilege
```

A useful experiment is:

```text
Authentication Attempt
        ↓
Windows / Linux Logs
        ↓
Log Collection
        ↓
Detection Rule
        ↓
Security Alert
        ↓
Investigation
        ↓
Remediation
```

---

# Safety & Authorization

This repository is intended for:

* Authorized penetration testing
* Security research
* CTF environments
* Personal laboratories
* Defensive security testing
* Cybersecurity education

Do **not** use these techniques against:

* Accounts you do not own
* Public login portals without permission
* Third-party infrastructure
* Production systems outside the authorized scope
* Stolen credentials
* Real-world targets without written authorization

Always define the testing scope before beginning an assessment.

---

# Directory Structure

```text
Attack Guide/
│
├── README.md
│
├── Hashcat.md
├── john-the-ripper.md
├── hashcat-utils.md
├── ophcrack.md
├── rainbowcrack.md
│
├── hydra.md
├── medusa.md
├── ncrack.md
├── patator.md
├── crowbar.md
│
└── crackmapexec.md
```

---

# Disclaimer

These guides are provided for educational and authorized security-testing purposes.

The tools documented here can perform authentication testing and security assessment activities that may cause account lockouts, service disruption, or security alerts when misused.

Always obtain explicit authorization before testing systems that you do not own.

**Learn responsibly. Test legally. Document everything.**
