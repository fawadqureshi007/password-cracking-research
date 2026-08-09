# CrackMapExec — Practical Guide

> Practical CrackMapExec reference for authorized Active Directory security assessments, Windows laboratory environments, CTFs, and defensive security research.

---

# 1. Overview

CrackMapExec (CME) is a network security assessment tool designed primarily for testing Windows and Active Directory environments.

It can help security professionals evaluate:

```text
SMB
WinRM
LDAP
MSSQL
RDP
Active Directory
Windows authentication
Network configuration
Credential exposure
```

Use it only against systems you own or are explicitly authorized to assess.

---

# 2. Install

### Kali Linux

Update package information:

```bash
sudo apt update
```

Search:

```bash
apt search crackmapexec
```

Depending on your Kali release, CME may be replaced or superseded by NetExec.

Check:

```bash
which crackmapexec
```

If available:

```bash
crackmapexec --help
```

---

# 3. NetExec Alternative

Modern environments may provide NetExec:

```bash
which nxc
```

Check:

```bash
nxc --help
```

The project evolved from CrackMapExec, so some environments use:

```text
nxc
```

instead of:

```text
crackmapexec
```

Always verify the tool installed on your system.

---

# 4. Check Version

CrackMapExec:

```bash
crackmapexec --version
```

Help:

```bash
crackmapexec --help
```

NetExec:

```bash
nxc --version
```

---

# 5. Available Protocols

Display help:

```bash
crackmapexec --help
```

Depending on the version, common protocol modules include:

```text
smb
winrm
ldap
mssql
rdp
ssh
ftp
```

Not every installation supports every protocol.

---

# 6. Create an Isolated Lab

A simple Windows laboratory can contain:

```text
Domain Controller
       |
       +--- Windows Client
       |
       +--- Windows Server
       |
       +--- Linux Testing Machine
```

Example private network:

```text
192.168.56.0/24
```

Example:

```text
DC01     192.168.56.10
WIN01    192.168.56.20
KALI     192.168.56.30
```

Use private laboratory addressing.

---

# 7. Test Connectivity

Check the target:

```bash
ping -c 3 192.168.56.10
```

Check SMB:

```bash
nc -vz 192.168.56.10 445
```

Check WinRM:

```bash
nc -vz 192.168.56.10 5985
```

Check LDAP:

```bash
nc -vz 192.168.56.10 389
```

---

# 8. SMB Help

```bash
crackmapexec smb --help
```

For NetExec:

```bash
nxc smb --help
```

The exact options can differ between versions.

---

# 9. SMB Host Identification

Against your laboratory server:

```bash
crackmapexec smb 192.168.56.10
```

Typical information can include:

```text
Hostname
Operating System
Domain
SMB signing
SMB protocol information
```

This is useful for initial laboratory enumeration.

---

# 10. SMB Network Range

For an isolated lab:

```bash
crackmapexec smb 192.168.56.0/24
```

This can help identify Windows systems exposing SMB.

Use only private laboratory ranges or explicitly authorized networks.

---

# 11. Multiple Targets

Create:

```bash
cat > targets.txt <<'EOF'
192.168.56.10
192.168.56.20
192.168.56.21
EOF
```

Depending on the installed version, supply targets according to its supported input syntax.

Check:

```bash
crackmapexec smb --help
```

---

# 12. SMB Authentication Validation

Create a dedicated laboratory account:

```text
Username:
labuser

Password:
LAB-TEST-PASSWORD
```

Then validate authentication against your test machine using the credentials supplied by your lab environment.

General structure:

```bash
crackmapexec smb <LAB-TARGET> -u <LAB-USER> -p '<LAB-PASSWORD>'
```

Do not place real production passwords into command history.

---

# 13. Avoid Shell History Exposure

Instead of putting a real password directly into a command:

```bash
history
```

may expose previously entered commands.

For sensitive authorized testing, use your tool's supported credential-input mechanisms and avoid storing real credentials in shell history.

---

# 14. Local Laboratory Account

If testing a local Windows account:

```text
WORKSTATION\labuser
```

or the appropriate local-account format supported by your tool version.

Validate only against the Windows VM you control.

---

# 15. Domain Laboratory Account

Example:

```text
LAB.LOCAL\labuser
```

Use the domain account created specifically for your Active Directory laboratory.

General structure:

```bash
crackmapexec smb <LAB-TARGET> -d LAB.LOCAL -u <LAB-USER> -p '<LAB-PASSWORD>'
```

---

# 16. SMB Signing Research

Run:

```bash
crackmapexec smb 192.168.56.10
```

Record whether SMB signing is:

```text
Enabled
Required
Not required
```

From a defensive perspective, SMB signing requirements can reduce certain classes of credential-relay risk.

---

# 17. SMB Security Assessment

Document:

```text
Host
Hostname
Domain
Operating System
SMB Version
Signing Status
Authentication Result
Observed Security Controls
```

Example:

```text
Host: 192.168.56.10
Hostname: DC01
Domain: LAB.LOCAL
SMB Signing: Required
Authentication: Successful
```

---

# 18. Windows Shares

In an authorized lab, enumerate accessible SMB shares using the supported SMB functionality.

General structure:

```bash
crackmapexec smb <LAB-TARGET> -u <LAB-USER> -p '<LAB-PASSWORD>' --shares
```

Check:

```bash
crackmapexec smb --help
```

because option names can vary between versions.

---

# 19. Share Review

Document:

```text
Share
Access Level
Purpose
Sensitive Data
Anonymous Access
Authenticated Access
```

Example:

```text
Share: Public
Read: Yes
Write: No
Sensitive Data: None
```

Avoid downloading real sensitive information during testing.

---

# 20. Defensive Share Audit

Look for:

```text
Everyone: Full Control
Authenticated Users: Write
Anonymous Access
Writable Administrative Shares
Sensitive Files
Backup Files
Configuration Files
```

Document excessive permissions as findings.

---

# 21. Windows Remote Management

Check WinRM:

```bash
nc -vz 192.168.56.10 5985
```

HTTPS WinRM:

```bash
nc -vz 192.168.56.10 5986
```

Check module help:

```bash
crackmapexec winrm --help
```

---

# 22. WinRM Authentication Validation

Use a dedicated laboratory account:

```bash
crackmapexec winrm <LAB-TARGET> \
  -u <LAB-USER> \
  -p '<LAB-PASSWORD>'
```

This validates whether the account is permitted to authenticate through WinRM.

---

# 23. WinRM Security Review

Document:

```text
HTTP WinRM
HTTPS WinRM
Authentication Method
Allowed Users
Network Restrictions
Logging
MFA / Conditional Access
```

The goal is to determine whether remote-management access is appropriately restricted.

---

# 24. LDAP

Check:

```bash
crackmapexec ldap --help
```

Connect to your laboratory domain controller:

```bash
crackmapexec ldap 192.168.56.10
```

Record:

```text
Hostname
Domain
LDAP availability
Directory information
Security configuration
```

---

# 25. LDAP Authentication

Use the dedicated laboratory account:

```bash
crackmapexec ldap <LAB-DC> \
  -d LAB.LOCAL \
  -u <LAB-USER> \
  -p '<LAB-PASSWORD>'
```

Only use credentials created for your assessment.

---

# 26. LDAP Security Review

Assess:

```text
LDAP Signing
LDAP Channel Binding
LDAPS
Anonymous LDAP Access
Directory Permissions
Authentication Policies
```

Record security weaknesses for remediation.

---

# 27. MSSQL

Check:

```bash
crackmapexec mssql --help
```

Check your laboratory SQL server:

```bash
crackmapexec mssql 192.168.56.20
```

Use a dedicated test database and account.

---

# 28. MSSQL Authentication Validation

General structure:

```bash
crackmapexec mssql <LAB-TARGET> \
  -u <LAB-USER> \
  -p '<LAB-PASSWORD>'
```

Document:

```text
SQL Server Version
Authentication Type
Account Permissions
Encryption
Network Exposure
```

---

# 29. RDP

Check:

```bash
crackmapexec rdp --help
```

Test whether the laboratory service is reachable:

```bash
nc -vz 192.168.56.20 3389
```

Review:

```text
Network Level Authentication
MFA
Account Restrictions
Firewall Rules
Logging
```

---

# 30. RDP Security Assessment

Document:

```text
RDP Enabled
Internet Exposure
NLA Enabled
Allowed Users
MFA
Lockout Policy
Network Restrictions
```

Never perform password attacks against public RDP services.

---

# 31. SSH

If your build supports SSH:

```bash
crackmapexec ssh --help
```

Check:

```bash
nc -vz 192.168.56.30 22
```

Use only your own laboratory Linux machine.

---

# 32. FTP

If supported:

```bash
crackmapexec ftp --help
```

Check:

```bash
nc -vz 192.168.56.30 21
```

Review:

```text
Anonymous Access
Authentication
Encryption
Directory Permissions
Logging
```

---

# 33. Host Inventory

Create:

```bash
cat > inventory.txt <<'EOF'
192.168.56.10 DC01
192.168.56.20 WIN01
192.168.56.30 KALI
EOF
```

Maintain an inventory containing:

```text
IP
Hostname
Operating System
Role
Owner
Testing Scope
```

---

# 34. Domain Inventory

Document your laboratory:

```text
Domain:
LAB.LOCAL

Domain Controller:
DC01

Clients:
WIN01
WIN02

Test Accounts:
labuser
audituser
```

Do not include production credentials in documentation.

---

# 35. Authentication Testing Matrix

Create:

```text
Target | Protocol | Account | Result | Notes
------------------------------------------------
DC01   | SMB      | labuser | PASS   | ...
WIN01  | WinRM    | labuser | PASS   | ...
DC01   | LDAP     | labuser | PASS   | ...
SQL01  | MSSQL    | audit  | PASS   | ...
```

This provides a repeatable assessment record.

---

# 36. Credential Validation

For authorized assessments, validate whether a test credential works only where it is expected to work.

Record:

```text
Credential
Expected Host
Actual Host
Protocol
Result
Security Impact
```

This helps identify:

```text
Password Reuse
Excessive Access
Unexpected Remote Access
Overprivileged Accounts
```

---

# 37. Password Reuse Research

Create several laboratory systems:

```text
WIN01
WIN02
WIN03
```

Create a dedicated test account with intentionally reused laboratory credentials.

Test the approved scope.

Document:

```text
Account
Host
Authentication Result
Expected Access
Unexpected Access
Remediation
```

The goal is to demonstrate why password reuse is dangerous.

---

# 38. Privilege Review

After authentication validation, document the account's intended privileges.

Example:

```text
Account:
labuser

Expected:
Standard User

Observed:
Administrative Access
```

This should become a security finding.

---

# 39. Local Administrator Review

For a controlled Windows machine, determine whether the laboratory account has unnecessary administrative privileges.

Document:

```text
Account
Host
Group Membership
Business Requirement
Recommended Permission
```

Follow least privilege.

---

# 40. Domain Administrator Risk

Never use a real Domain Administrator account for routine testing.

Create a dedicated assessment account instead.

Document:

```text
Account
Group Membership
Required Privileges
Actual Privileges
Risk
Remediation
```

High-privilege credentials should be tightly controlled.

---

# 41. Credential Hygiene

Check the laboratory environment for:

```text
Shared Passwords
Password Reuse
Weak Passwords
Default Credentials
Expired Accounts
Unused Accounts
Service Accounts
Excessive Privileges
```

Remediation should include:

```text
Unique Passwords
MFA
Password Managers
Least Privilege
Account Lifecycle Management
Monitoring
```

---

# 42. Logging

Monitor the Windows test system while performing authentication validation.

Useful Windows Event Logs include:

```text
Security
System
Windows PowerShell
Microsoft-Windows-WinRM
```

Look for:

```text
Successful Logon
Failed Logon
Remote Logon
Account Lockout
Privilege Use
```

---

# 43. Failed Authentication Research

Create a dedicated test account.

Perform a small number of intentionally incorrect authentication attempts.

Monitor:

```text
Event ID
Source Address
Username
Timestamp
Authentication Protocol
Failure Reason
```

Use this to evaluate detection and alerting.

---

# 44. Account Lockout Testing

Define the laboratory policy first:

```text
Threshold:
5 attempts

Lockout:
10 minutes
```

Use a controlled test account.

Stop immediately if the environment behaves unexpectedly.

Record:

```text
Attempt Number
Lockout Trigger
Lockout Duration
Event Log
Alert
Recovery
```

---

# 45. Detection Engineering

Build a detection workflow:

```text
Authentication Attempt
        ↓
Windows Event Log
        ↓
Log Collection
        ↓
Detection Rule
        ↓
Alert
        ↓
Investigation
```

Useful signals include:

```text
Repeated Failed Logons
Multiple Hosts
Multiple Usernames
Unusual Source Host
Successful Login After Failures
Privileged Account Login
```

---

# 46. SMB Security Checklist

Review:

```text
[ ] SMBv1 Disabled
[ ] SMB Signing Required
[ ] Administrative Shares Restricted
[ ] Firewall Enabled
[ ] Guest Access Disabled
[ ] Anonymous Access Disabled
[ ] Least Privilege Applied
[ ] Sensitive Shares Reviewed
[ ] Authentication Logging Enabled
```

---

# 47. WinRM Security Checklist

Review:

```text
[ ] WinRM Required
[ ] HTTPS Considered
[ ] Trusted Hosts Restricted
[ ] Firewall Rules Restricted
[ ] Administrative Access Limited
[ ] Logging Enabled
[ ] MFA / Additional Controls
[ ] Unused Remote Management Disabled
```

---

# 48. LDAP Security Checklist

Review:

```text
[ ] LDAP Signing
[ ] Channel Binding
[ ] LDAPS
[ ] Anonymous Access Disabled
[ ] Directory Permissions Reviewed
[ ] Privileged Groups Audited
[ ] Authentication Logs Enabled
```

---

# 49. RDP Security Checklist

Review:

```text
[ ] NLA Enabled
[ ] RDP Not Internet Exposed
[ ] MFA
[ ] Firewall Restrictions
[ ] Restricted Users
[ ] Account Lockout
[ ] Logging
[ ] Session Controls
```

---

# 50. Reporting

Create:

```bash
mkdir -p results
nano results/report.md
```

Use:

```text
# Windows Security Assessment

## Scope

Systems:
[LAB SYSTEMS]

Protocols:
SMB / WinRM / LDAP / RDP / MSSQL

## Findings

### Finding 1
Title:
Severity:
Affected Host:
Evidence:
Impact:
Recommendation:

### Finding 2
Title:
Severity:
Affected Host:
Evidence:
Impact:
Recommendation:
```

Never include real passwords in the report.

---

# 51. Evidence Collection

Record:

```text
Timestamp
Target
Protocol
Tool Version
Command Category
Result
Screenshot
Relevant Event Log
```

Redact:

```text
Passwords
Tokens
Private Keys
Session Cookies
Personal Information
```

---

# 52. Tool Version Recording

Before an assessment:

```bash
crackmapexec --version
```

or:

```bash
nxc --version
```

Record:

```text
Tool:
Version:
Operating System:
Date:
Assessment Scope:
```

This makes results reproducible.

---

# 53. Troubleshooting

### Command Not Found

```bash
which crackmapexec
```

Then:

```bash
which nxc
```

If `nxc` exists, your environment may be using the modern replacement.

---

### Target Unreachable

```bash
ping -c 3 192.168.56.10
```

Check SMB:

```bash
nc -vz 192.168.56.10 445
```

Check firewall and routing.

---

### Authentication Failure

Verify manually that:

```text
Username
Password
Domain
Target
Protocol
```

are correct for the laboratory.

Do not immediately increase authentication attempts.

---

### Access Denied

Determine whether the account is actually authorized for the requested resource.

Review:

```text
Local Groups
Domain Groups
Share Permissions
NTFS Permissions
Remote Access Policies
```

---

### Module Not Available

Check:

```bash
crackmapexec --help
```

or:

```bash
nxc --help
```

Module availability depends on the installed version.

---

# 54. Practical Progression

```text
Level 1
Installation
Version
Help
Lab Networking

        ↓

Level 2
SMB Enumeration
Host Identification
Signing Review
Share Auditing

        ↓

Level 3
Credential Validation
WinRM
LDAP
MSSQL
RDP

        ↓

Level 4
Active Directory Review
Privilege Analysis
Password Reuse Research
Remote Access Review

        ↓

Level 5
Windows Event Logs
Detection Engineering
Account Lockout Research
SIEM Correlation

        ↓

Level 6
Complete AD Security Assessment
Findings
Evidence
Remediation
Repeatable Reporting
```

---

# 55. Quick Reference

```bash
# Help
crackmapexec --help

# Version
crackmapexec --version

# SMB help
crackmapexec smb --help

# WinRM help
crackmapexec winrm --help

# LDAP help
crackmapexec ldap --help

# MSSQL help
crackmapexec mssql --help

# RDP help
crackmapexec rdp --help

# SMB host identification
crackmapexec smb 192.168.56.10

# SMB lab network
crackmapexec smb 192.168.56.0/24

# Lab credential validation
crackmapexec smb <LAB-TARGET> \
  -d <LAB-DOMAIN> \
  -u <LAB-USER> \
  -p '<LAB-PASSWORD>'

# Check WinRM
nc -vz 192.168.56.10 5985

# Check SMB
nc -vz 192.168.56.10 445

# Check LDAP
nc -vz 192.168.56.10 389

# Check RDP
nc -vz 192.168.56.10 3389

# Modern replacement, where installed
nxc --help
nxc smb --help
```

---

# 56. Final Laboratory Workflow

```text
Define Authorization
        ↓
Build Isolated Windows Lab
        ↓
Identify Hosts
        ↓
Identify Services
        ↓
Review SMB / LDAP / WinRM / RDP
        ↓
Create Dedicated Test Accounts
        ↓
Validate Expected Authentication
        ↓
Review Permissions
        ↓
Monitor Windows Logs
        ↓
Test Security Controls
        ↓
Document Findings
        ↓
Recommend Remediation
        ↓
Retest
        ↓
Finalize Report
```

---

# Scope

CrackMapExec is primarily a Windows and Active Directory security-assessment framework rather than a standalone password-cracking tool.

Use it only against:

* Your own Windows systems
* Isolated Active Directory laboratories
* CTF environments
* Explicitly authorized penetration-testing targets

Do not use credential-testing, enumeration, or remote-access functionality against third-party systems without authorization.

Never publish real passwords, hashes, private keys, authentication tokens, or sensitive domain information.
