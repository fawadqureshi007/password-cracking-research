# 🔐 Password Cracking Research

<p align="center">
  <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=28&duration=2800&pause=900&color=00FF88&center=true&vCenter=true&width=850&lines=PASSWORD+SECURITY+RESEARCH;HASH+RECOVERY+%7C+AUTHENTICATION+AUDITING;OFFLINE+%26+NETWORK+SECURITY+RESEARCH;LEARN+%E2%80%A2+TEST+%E2%80%A2+ANALYZE+%E2%80%A2+DEFEND" alt="Typing Animation">
</p>

<p align="center">
  <b>A practical cybersecurity research repository focused on password security, hash recovery, authentication auditing, and controlled security testing.</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Focus-Password%20Security-111827?style=for-the-badge">
  <img src="https://img.shields.io/badge/Research-Authorized%20Testing-111827?style=for-the-badge">
  <img src="https://img.shields.io/badge/Platform-Linux%20%7C%20Windows-111827?style=for-the-badge">
  <img src="https://img.shields.io/badge/Status-Active-111827?style=for-the-badge">
</p>

---

## ⚡ What Is This?

**Password Cracking Research** is a practical reference and laboratory-focused repository for understanding how password-security systems are evaluated.

The project covers both **offline password recovery** and **online authentication-security testing**, while also examining the defensive controls that make these attacks harder to perform.

```text
                     PASSWORD SECURITY RESEARCH
                               │
             ┌─────────────────┴─────────────────┐
             │                                   │
       OFFLINE RESEARCH                    ONLINE RESEARCH
             │                                   │
       ┌─────┴─────┐                     ┌───────┴────────┐
       │           │                     │                │
     Hashes     Recovery            Authentication    Services
       │           │                     │                │
       ▼           ▼                     ▼                ▼
   Hashcat      John                  Hydra            Medusa
   Ophcrack     RainbowCrack          Ncrack           Patator
   Hashcat      Utilities             Crowbar          Network Tests
             │
             ▼
      WINDOWS / AD RESEARCH
             │
             ▼
        CrackMapExec
```

---

# 🎯 Objectives

This repository is designed to help researchers understand:

* How password hashes are analyzed
* How password candidates are generated
* How different recovery strategies work
* How authentication services can be security-tested
* How rate limiting affects authentication attempts
* How account lockout policies behave
* How Windows and Active Directory authentication can be assessed
* How defensive monitoring detects suspicious authentication activity
* How to document repeatable security experiments

The goal is **understanding password security**, not simply running tools.

---

# 🧠 Core Research Areas

```text
┌──────────────────────────────────────────────────────────────┐
│                    PASSWORD SECURITY                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Hash Analysis        Candidate Generation                   │
│  Password Recovery    Authentication Testing                 │
│  Wordlists            Mask Techniques                         │
│  Rules                Brute-Force Research                    │
│  Rainbow Tables       Windows Security                        │
│  Active Directory     Detection Engineering                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

# 📚 Repository Structure

```text
password-cracking-research/
│
├── 📁 Foundations/
│   └── 📄 Cracking-Time.md
│
├── 📁 Attack Guide/
│   │
│   ├── 📄 README.md
│   │
│   ├── ⚡ Hashcat.md
│   ├── 🔨 john-the-ripper.md
│   ├── 🌐 hydra.md
│   ├── 🔑 medusa.md
│   ├── 🛡️ ncrack.md
│   ├── ⚙️ patator.md
│   ├── 🔐 crowbar.md
│   ├── 🧰 hashcat-utils.md
│   ├── 🪟 ophcrack.md
│   ├── 🌈 rainbowcrack.md
│   └── 🖥️ crackmapexec.md
│
├── 📁 Attack Techniques/
│   ├── 📄 README.md
│   ├── 📄 dictionary-attacks.md
│   ├── 📄 brute-force-attacks.md
│   ├── 📄 mask-attacks.md
│   └── 📄 incremental-attacks.md
│
├── 📁 Password Recovery/
│   ├── 📄 office-password-recovery.md
│   ├── 📄 archive-password-recovery.md
│   ├── 📄 pdf-password-recovery.md
│   ├── 📄 database-password-recovery.md
│   ├── 📄 browser-password-recovery.md
│   ├── 📄 cloud-password-recovery.md
│   ├── 📄 ssh-key-recovery.md
│   └── 📄 password-manager-recovery.md
│
├── 📄 README.md
├── 📄 LICENSE
└── 📄 .gitignore
```

---

# 🛠️ Tool Collection

## ⚡ Hashcat

GPU-accelerated password-recovery and hash-analysis framework.

```text
Hash Modes
Wordlists
Mask Attacks
Rules
Charsets
Benchmarking
Candidate Generation
Recovery Workflows
```

→ [`Attack Guide/Hashcat.md`](Attack%20Guide/Hashcat.md)

---

## 🔨 John the Ripper

A powerful password-security auditing and offline hash-recovery framework.

```text
Hash Identification
Wordlists
Rules
Incremental Modes
Custom Candidates
Session Management
Recovery
```

→ [`Attack Guide/john-the-ripper.md`](Attack%20Guide/john-the-ripper.md)

---

## 🌐 Hydra

Parallelized network authentication-security testing framework.

```text
SSH
FTP
HTTP
HTTPS
SMTP
IMAP
POP3
Database Services
RDP
SMB
```

→ [`Attack Guide/hydra.md`](Attack%20Guide/hydra.md)

---

## 🔑 Medusa

Parallel network-login auditing framework designed for controlled authentication testing.

```text
Authentication Modules
Credential Auditing
Network Services
Laboratory Testing
Logging
Rate-Limit Research
```

→ [`Attack Guide/medusa.md`](Attack%20Guide/medusa.md)

---

## 🛡️ Ncrack

Network authentication auditing tool for controlled security assessments.

```text
Network Services
Authentication Testing
Credential Auditing
Service Analysis
Laboratory Research
```

→ [`Attack Guide/ncrack.md`](Attack%20Guide/ncrack.md)

---

## ⚙️ Patator

Modular framework for authentication and protocol testing.

```text
Module Architecture
Request Testing
Credential Validation
Protocol Research
Laboratory Automation
```

→ [`Attack Guide/patator.md`](Attack%20Guide/patator.md)

---

## 🔐 Crowbar

Credential-testing tool supporting selected remote authentication technologies.

```text
RDP
SSH Keys
OpenVPN
Credential Auditing
Controlled Authentication Testing
```

→ [`Attack Guide/crowbar.md`](Attack%20Guide/crowbar.md)

---

## 🧰 Hashcat Utils

Utilities for preparing and processing password candidates.

```text
Candidate Generation
Wordlist Processing
Charset Operations
Password Data Transformation
Hashcat Workflows
```

→ [`Attack Guide/hashcat-utils.md`](Attack%20Guide/hashcat-utils.md)

---

## 🪟 Ophcrack

Windows password-recovery research tool using rainbow-table techniques.

```text
NTLM Research
Rainbow Tables
Windows Password Auditing
Offline Recovery
```

→ [`Attack Guide/ophcrack.md`](Attack%20Guide/ophcrack.md)

---

## 🌈 RainbowCrack

Rainbow-table-based password-recovery research.

```text
Table Generation
Hash Generation
Table Lookup
Password Recovery
```

→ [`Attack Guide/rainbowcrack.md`](Attack%20Guide/rainbowcrack.md)

---

## 🖥️ CrackMapExec

Windows and Active Directory security-assessment framework.

```text
SMB
WinRM
LDAP
MSSQL
RDP
Windows Hosts
Active Directory
Credential Validation
Security Configuration
```

> CrackMapExec is primarily a Windows/Active Directory assessment tool rather than a standalone password-cracking utility.

→ [`Attack Guide/crackmapexec.md`](Attack%20Guide/crackmapexec.md)

---

# 🧪 Attack Techniques

The repository also documents the underlying concepts behind password-recovery techniques.

| Technique      | Purpose                                      |
| -------------- | -------------------------------------------- |
| Dictionary     | Test candidates from known wordlists         |
| Brute Force    | Explore a defined candidate space            |
| Mask           | Generate candidates matching a known pattern |
| Incremental    | Progressively explore candidate combinations |
| Rule-Based     | Transform existing password candidates       |
| Rainbow Tables | Precomputed hash-recovery research           |

Explore:

→ [`Attack Techniques/`](Attack%20Techniques/)

---

# 🔐 Password Recovery Research

The repository includes research covering different password-protected formats and credential stores.

```text
Office Documents
        │
        ├── Word
        ├── Excel
        └── PowerPoint

Archives
        │
        ├── ZIP
        ├── 7z
        └── RAR

Documents
        │
        └── PDF

Credential Stores
        │
        ├── Databases
        ├── Browsers
        ├── Password Managers
        └── SSH Keys
```

Explore:

→ [`Password Recovery/`](Password%20Recovery/)

---

# ⏱️ Understanding Cracking Time

Password recovery is not simply about running a faster tool.

Recovery time depends on:

```text
Candidate Space
      ×
Hash Algorithm
      ×
Hardware
      ×
Candidate Generation
      ×
Implementation
      ×
Password Structure
```

Research should therefore measure:

```text
Hash Type
Candidate Count
Candidate Rate
Hardware
Attack Strategy
Elapsed Time
Recovery Result
```

See:

→ [`Foundations/`](Foundations/)

---

# 🧭 Recommended Learning Path

### Level 01 — Foundations

```text
Password Entropy
      ↓
Hashing
      ↓
Salting
      ↓
Hash Identification
      ↓
Cracking-Time Models
```

### Level 02 — Offline Recovery

```text
Hashcat
      ↓
John the Ripper
      ↓
Wordlists
      ↓
Rules
      ↓
Masks
      ↓
Incremental Research
```

### Level 03 — Authentication Testing

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

### Level 04 — Specialized Recovery

```text
Ophcrack
      ↓
RainbowCrack
      ↓
Hashcat Utils
```

### Level 05 — Windows / Active Directory

```text
SMB
 ↓
Windows Authentication
 ↓
WinRM
 ↓
LDAP
 ↓
Active Directory
 ↓
Security Assessment
```

---

# 🔬 Research Methodology

Every experiment should follow a repeatable process:

```text
┌─────────────────────┐
│ DEFINE OBJECTIVE    │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ DEFINE AUTHORIZED   │
│ SCOPE               │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ BUILD LAB           │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ CREATE TEST DATA    │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ SELECT TECHNIQUE    │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ RUN CONTROLLED TEST │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ MONITOR LOGS        │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ ANALYZE RESULTS     │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ DOCUMENT FINDINGS   │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ REMEDIATE + RETEST  │
└─────────────────────┘
```

---

# 🧰 Recommended Laboratory

A practical isolated environment can look like:

```text
                         ┌─────────────────┐
                         │   TEST NETWORK  │
                         └────────┬────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
              ▼                   ▼                   ▼
        ┌───────────┐       ┌───────────┐       ┌───────────┐
        │   Kali    │       │ Windows   │       │    AD     │
        │  /Parrot  │       │    VM     │       │   Lab     │
        └───────────┘       └───────────┘       └───────────┘
              │                   │                   │
              └───────────────────┴───────────────────┘
                                  │
                           Controlled Testing
```

Recommended components:

```text
Linux Security VM
Windows Test VM
Windows Server
Active Directory Domain
Dedicated Test Accounts
Test Password Dataset
Isolated Network
Centralized Logging
```

---

# 📊 What To Measure

For serious research, record more than simply whether a password was recovered.

```text
┌─────────────────────────────┐
│ Experiment                  │
├─────────────────────────────┤
│ Tool                        │
│ Version                     │
│ Hash / Protocol             │
│ Hardware                    │
│ Candidate Dataset           │
│ Candidate Count             │
│ Candidate Rate              │
│ Threads / Workload          │
│ Start Time                  │
│ End Time                    │
│ Recovery Result             │
│ Detection Result            │
└─────────────────────────────┘
```

This makes experiments reproducible.

---

# 🛡️ Defensive Research

Password-security research should also evaluate defenses.

Important areas include:

```text
Strong Password Policies
Password Managers
Unique Credentials
MFA
Account Lockout
Rate Limiting
Secure Hashing
Salting
Key Stretching
Credential Monitoring
Authentication Logging
SIEM Detection
Least Privilege
```

A useful defensive experiment:

```text
Test Authentication
        ↓
Generate Authentication Events
        ↓
Collect Logs
        ↓
Detect Pattern
        ↓
Generate Alert
        ↓
Investigate
        ↓
Remediate
        ↓
Retest
```

---

# 🚨 Security Considerations

Password and authentication-testing tools can generate:

```text
Account Lockouts
Authentication Alerts
Service Load
Network Traffic
Security Events
Potential Data Exposure
```

Always begin with a small controlled test.

Never assume that a tool is harmless simply because it is publicly available.

---

# ⚖️ Responsible Use

This repository is intended for:

* Cybersecurity education
* Authorized penetration testing
* Personal laboratories
* CTF environments
* Password-security research
* Defensive security research
* Security-tool experimentation

Do **not** use the material against systems, accounts, or networks without explicit authorization.

Never test:

```text
Third-Party Accounts
Public Login Portals
Production Systems Outside Scope
Stolen Credentials
Unauthorized Networks
```

---

# 🧹 Credential Hygiene

Never commit sensitive information to this repository.

Do not upload:

```text
Real Passwords
Password Hashes
Private Keys
API Keys
Session Tokens
Cookies
Production Credentials
Personal Data
Internal Host Information
```

Use synthetic laboratory data instead.

---

# 📝 Reporting Template

A useful security finding should contain:

```text
Title
Severity
Affected System
Affected Account
Description
Technical Evidence
Security Impact
Root Cause
Recommendation
Retest Result
```

Example:

```text
Finding:
Weak Authentication Controls

Impact:
Repeated authentication attempts were not sufficiently
rate-limited in the laboratory environment.

Recommendation:
Implement rate limiting, account lockout controls,
monitoring, and MFA where appropriate.
```

---

# 🚀 Project Philosophy

```text
Learn
  ↓
Understand
  ↓
Experiment
  ↓
Measure
  ↓
Analyze
  ↓
Defend
  ↓
Improve
```

The objective is not simply to recover a password.

The objective is to understand **why the password was recoverable, what made it difficult or easy, how the system responded, and how the security posture can be improved.**

---

# 📂 Quick Navigation

| Section                                      | Description                        |
| -------------------------------------------- | ---------------------------------- |
| [`Foundations/`](Foundations/)               | Core password-security concepts    |
| [`Attack Guide/`](Attack%20Guide/)           | Practical security-tool references |
| [`Attack Techniques/`](Attack%20Techniques/) | Password attack methodology        |
| [`Password Recovery/`](Password%20Recovery/) | Format-specific recovery research  |

---

# ⭐ Repository Focus

```text
                ┌───────────────────────┐
                │ PASSWORD SECURITY     │
                └───────────┬───────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
     OFFLINE             ONLINE             WINDOWS
     RECOVERY           TESTING                / AD
        │                   │                   │
        ▼                   ▼                   ▼
     HASHES           AUTHENTICATION         SMB
     WORDLISTS        SERVICES               LDAP
     MASKS            PROTOCOLS              WINRM
     RULES            RATE LIMITS             RDP
     TABLES           LOCKOUTS               MSSQL
        │                   │                   │
        └───────────────────┼───────────────────┘
                            ▼
                     SECURITY RESEARCH
                            │
                            ▼
                      DEFENSIVE IMPROVEMENT
```

---

# 📈 Project Status

```text
Foundations              ████████████████████  Complete
Attack Guide             ████████████████████  Complete
Attack Techniques        ████████████████████  Complete
Password Recovery        ████████████████████  Complete
Documentation            ████████████████████  Active
Research                 ███████████████████░  Ongoing
```

---

# 🤝 Contributing

Contributions are welcome when they improve the educational and research value of the project.

Useful contributions include:

```text
New Security Research
Improved Documentation
Lab Methodologies
Detection Techniques
Defensive Analysis
Tool Corrections
Reproducible Experiments
```

Please avoid submitting:

```text
Real Credentials
Unauthorized Targets
Stolen Data
Sensitive Information
Malicious Infrastructure
```

---

# 📜 License

See [`LICENSE`](LICENSE) for the repository license.

---

# ⚠️ Disclaimer

This repository is provided for **educational, research, and authorized security-testing purposes**.

The author does not encourage unauthorized access, credential theft, account compromise, or attacks against systems without permission.

You are responsible for ensuring that your use of these tools and techniques complies with applicable laws, policies, and authorization requirements.

---

<p align="center">

<b>🔐 PASSWORD SECURITY RESEARCH</b>

<br>

<code>LEARN • TEST • ANALYZE • DEFEND</code>

<br><br>

<i>Build knowledge. Understand the attack. Improve the defense.</i>

</p>
---

<p align="center">
  <b>© 2026 Fawad Qureshi</b><br>
  Security Researcher • <a href="https://instagram.com/h4cker_fawad">@h4cker_fawad</a><br>
  All rights reserved.
</p>
