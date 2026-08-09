# Crowbar — Practical Guide

> Practical Crowbar reference for authorized authentication-security testing, CTFs, and isolated laboratories.

---

## 1. Install

### Kali Linux

```bash
sudo apt update
sudo apt install crowbar
```

Check:

```bash
crowbar --help
```

Version/package information:

```bash
apt show crowbar
```

---

# 2. Basic Syntax

General structure:

```bash
crowbar [options]
```

Crowbar is commonly used for authentication testing against services such as:

```text
RDP
SSH
VNC
OpenVPN
```

Check supported options:

```bash
crowbar --help
```

---

# 3. Help

```bash
crowbar --help
```

For the installed version:

```bash
crowbar -h
```

Always check the local help because available options can differ between distributions and versions.

---

# 4. Create a Laboratory Password List

```bash
cat > passwords.txt <<'EOF'
test
test123
lab
lab123
research
research123
Password123
EOF
```

Check:

```bash
cat passwords.txt
```

Count:

```bash
wc -l passwords.txt
```

---

# 5. Create a Laboratory Username List

```bash
cat > users.txt <<'EOF'
labuser
testuser
research
EOF
```

Check:

```bash
cat users.txt
```

Count:

```bash
wc -l users.txt
```

---

# 6. RDP

Crowbar can be used for RDP authentication testing against a Windows laboratory VM.

Check the RDP service:

```bash
nc -vz 192.168.56.101 3389
```

Display help:

```bash
crowbar --help
```

A typical laboratory RDP password test uses:

```bash
crowbar \
  -b rdp \
  -s 192.168.56.101/32 \
  -u labuser \
  -C passwords.txt
```

Only use a private laboratory address or an explicitly authorized target.

---

# 7. RDP Username List

Create:

```bash
cat > users.txt <<'EOF'
labuser
testuser
research
EOF
```

Depending on the Crowbar version, username-file support can be checked with:

```bash
crowbar --help
```

Use the option documented by your installed version for supplying multiple usernames.

---

# 8. RDP Custom Port

If your laboratory RDP service uses a non-standard port, first inspect:

```bash
crowbar --help
```

Check connectivity:

```bash
nc -vz 192.168.56.101 3390
```

Then use the port option supported by your installed Crowbar version.

Do not assume every version accepts identical port syntax.

---

# 9. RDP Password Testing

Basic structure:

```bash
crowbar \
  -b rdp \
  -s 192.168.56.101/32 \
  -u labuser \
  -C passwords.txt
```

Use a deliberately created laboratory account.

Example test dataset:

```text
labuser
```

with:

```text
lab123
```

as the known laboratory password.

---

# 10. RDP Known-Credential Verification

Before automated testing, verify the account manually.

From a Linux system with an RDP client:

```bash
xfreerdp /v:192.168.56.101 /u:labuser
```

Enter the known laboratory password.

If manual authentication fails, fix the Windows laboratory configuration before troubleshooting Crowbar.

---

# 11. RDP Service Verification

Check the port:

```bash
nc -vz 192.168.56.101 3389
```

Scan the isolated VM:

```bash
nmap -sV -p 3389 192.168.56.101
```

Expected result:

```text
3389/tcp open  ms-wbt-server
```

---

# 12. SSH

Crowbar can also perform SSH authentication testing in supported configurations.

Check:

```bash
crowbar --help
```

Look for the SSH backend:

```text
ssh
```

Verify SSH:

```bash
nc -vz 192.168.56.102 22
```

Use the SSH options documented by your installed Crowbar version.

---

# 13. SSH Key Testing

Crowbar is particularly useful for testing SSH private-key authentication.

Create a dedicated laboratory key:

```bash
ssh-keygen \
  -t ed25519 \
  -f lab_id_ed25519
```

Do not use your personal SSH private key for testing.

List:

```bash
ls -l lab_id_ed25519*
```

---

# 14. SSH Public Key Deployment

Copy the public key to your laboratory account:

```bash
ssh-copy-id \
  -i lab_id_ed25519.pub \
  labuser@192.168.56.102
```

Test manually:

```bash
ssh \
  -i lab_id_ed25519 \
  labuser@192.168.56.102
```

If successful, the key is ready for controlled testing.

---

# 15. SSH Key List

Create a laboratory directory:

```bash
mkdir -p ssh-keys
```

Move test keys:

```bash
mv lab_id_ed25519 ssh-keys/
```

Create additional dedicated test keys only when required.

Never place real production private keys into a testing directory.

---

# 16. SSH Key Permission

Private keys should have restricted permissions:

```bash
chmod 600 ssh-keys/lab_id_ed25519
```

Check:

```bash
ls -l ssh-keys/
```

Expected:

```text
-rw------- ... lab_id_ed25519
```

---

# 17. VNC

Check the laboratory VNC service:

```bash
nc -vz 192.168.56.103 5900
```

Scan:

```bash
nmap -sV -p 5900 192.168.56.103
```

Check Crowbar support:

```bash
crowbar --help
```

Look for:

```text
vnc
```

Use the VNC backend and options documented by your installed version.

---

# 18. VNC Password List

Create a small laboratory dataset:

```bash
cat > vnc-passwords.txt <<'EOF'
test
lab123
research123
VNC123
EOF
```

Use the VNC backend according to:

```bash
crowbar --help
```

Only test a VNC server you own or are explicitly authorized to assess.

---

# 19. OpenVPN

Check available backends:

```bash
crowbar --help
```

Look for:

```text
openvpn
```

For a laboratory VPN environment, prepare:

```text
VPN configuration
Test username
Test password
Test certificate
```

Then use the syntax documented by your installed Crowbar version.

---

# 20. Backend Selection

Crowbar uses a backend option.

General pattern:

```bash
crowbar -b <backend> ...
```

Examples:

```text
-b rdp
-b ssh
-b vnc
-b openvpn
```

Check available functionality:

```bash
crowbar --help
```

---

# 21. Target Specification

Crowbar commonly uses the `-s` option for specifying targets.

Single laboratory host:

```bash
-s 192.168.56.101/32
```

Laboratory subnet:

```bash
-s 192.168.56.0/24
```

Only include systems that are part of your authorized test network.

---

# 22. Single Username

Example:

```bash
crowbar \
  -b rdp \
  -s 192.168.56.101/32 \
  -u labuser \
  -C passwords.txt
```

The test uses:

```text
Target: 192.168.56.101
Username: labuser
Password file: passwords.txt
```

---

# 23. Password File

Create:

```bash
cat > passwords.txt <<'EOF'
wrong1
wrong2
lab123
wrong3
EOF
```

Inspect:

```bash
cat passwords.txt
```

Run the controlled test:

```bash
crowbar \
  -b rdp \
  -s 192.168.56.101/32 \
  -u labuser \
  -C passwords.txt
```

---

# 24. Small Test First

Do not begin with a large wordlist.

Create:

```bash
cat > small.txt <<'EOF'
wrong1
wrong2
lab123
EOF
```

Run:

```bash
crowbar \
  -b rdp \
  -s 192.168.56.101/32 \
  -u labuser \
  -C small.txt
```

This makes troubleshooting easier.

---

# 25. Generate Numeric Laboratory Candidates

```bash
seq -w 0000 9999 > numbers.txt
```

Check:

```bash
head numbers.txt
```

Count:

```bash
wc -l numbers.txt
```

Use only against a dedicated laboratory account.

---

# 26. Generate Pattern-Based Candidates

Suppose the laboratory password format is:

```text
LAB + four digits
```

Generate:

```bash
for i in $(seq -w 0000 9999); do
    echo "LAB$i"
done > lab-passwords.txt
```

Check:

```bash
head lab-passwords.txt
```

Count:

```bash
wc -l lab-passwords.txt
```

---

# 27. Output

Crowbar reports authentication results during execution.

For repeatable experiments, capture terminal output:

```bash
crowbar \
  -b rdp \
  -s 192.168.56.101/32 \
  -u labuser \
  -C small.txt \
  2>&1 | tee rdp-test.log
```

Review:

```bash
less rdp-test.log
```

---

# 28. Experiment Directory

Create:

```bash
mkdir -p experiments
```

Run:

```bash
crowbar \
  -b rdp \
  -s 192.168.56.101/32 \
  -u labuser \
  -C small.txt \
  2>&1 | tee experiments/rdp-test-01.log
```

List:

```bash
ls -lah experiments/
```

---

# 29. Multiple Targets

Create an isolated target list:

```text
192.168.56.101
192.168.56.102
192.168.56.103
```

Check Crowbar's target-file support:

```bash
crowbar --help
```

Use only systems belonging to your laboratory.

---

# 30. RDP Laboratory Network

A simple lab can contain:

```text
Kali Linux
192.168.56.10
      |
      |
      +------ Windows VM
              192.168.56.101
              TCP/3389
```

Verify:

```bash
ping -c 2 192.168.56.101
```

Then:

```bash
nc -vz 192.168.56.101 3389
```

---

# 31. SSH Laboratory Network

Example:

```text
Kali Linux
192.168.56.10
      |
      |
      +------ Linux VM
              192.168.56.102
              TCP/22
```

Verify:

```bash
nc -vz 192.168.56.102 22
```

Manual login:

```bash
ssh labuser@192.168.56.102
```

---

# 32. VNC Laboratory Network

Example:

```text
Kali Linux
192.168.56.10
      |
      |
      +------ VNC VM
              192.168.56.103
              TCP/5900
```

Verify:

```bash
nc -vz 192.168.56.103 5900
```

---

# 33. Service Discovery

Before selecting a Crowbar backend:

```bash
nmap -sV 192.168.56.101
```

For specific services:

```bash
nmap -sV -p 22,3389,5900 192.168.56.101
```

Map the service to the appropriate backend.

---

# 34. RDP Logs

On a Windows laboratory machine, inspect authentication events using Event Viewer.

Relevant areas include:

```text
Windows Logs
    ↓
Security
```

Look for:

```text
Failed logons
Successful logons
Account name
Source address
Timestamp
```

Correlate these events with the Crowbar test.

---

# 35. SSH Logs

On the Linux laboratory server:

```bash
sudo journalctl -u ssh -f
```

Or:

```bash
sudo tail -f /var/log/auth.log
```

Run the Crowbar test from another terminal.

Record:

```text
Timestamp
Username
Source address
Authentication result
```

---

# 36. VNC Logs

VNC logging depends on the server implementation.

Inspect:

```bash
sudo journalctl -f
```

Search:

```bash
sudo journalctl | grep -i vnc
```

Locate application-specific logs when required.

---

# 37. Rate-Limit Research

Create:

```bash
cat > rate-test.txt <<'EOF'
wrong01
wrong02
wrong03
wrong04
wrong05
EOF
```

Run a controlled test.

Record:

```text
Attempts
Duration
Response time
Connection failures
Lockout
Server resource usage
```

Do not overwhelm the laboratory service.

---

# 38. Account-Lockout Research

Create a dedicated test account.

Use intentionally incorrect credentials:

```bash
cat > lockout-test.txt <<'EOF'
incorrect01
incorrect02
incorrect03
incorrect04
incorrect05
EOF
```

Run:

```bash
crowbar \
  -b rdp \
  -s 192.168.56.101/32 \
  -u labuser \
  -C lockout-test.txt
```

Monitor Windows authentication events.

Record:

```text
Attempts before lockout
Lockout duration
Event ID
Source address
Recovery process
```

---

# 39. Detection Testing

Build the experiment:

```text
Crowbar
   ↓
Authentication Attempts
   ↓
Windows/Linux Logs
   ↓
Detection Rule
   ↓
Alert
```

Test whether your monitoring detects:

```text
Repeated failures
Rapid failures
Multiple usernames
Repeated source address
Successful login after failures
```

---

# 40. Credential Policy Testing

Create several dedicated laboratory accounts.

Example:

```text
lab-weak
lab-medium
lab-strong
```

Test the configured authentication policy.

Measure:

```text
Lockout threshold
Lockout duration
Password complexity
Authentication delay
Logging
Alerting
```

Never test against real user accounts.

---

# 41. SSH Key Research

Generate a dedicated key:

```bash
ssh-keygen \
  -t ed25519 \
  -f lab_key
```

Check:

```bash
ls -l lab_key*
```

Protect:

```bash
chmod 600 lab_key
```

Test:

```bash
ssh \
  -i lab_key \
  labuser@192.168.56.102
```

---

# 42. SSH Key Inventory

Create:

```bash
mkdir -p ssh-keys
```

Store only laboratory keys:

```bash
mv lab_key ssh-keys/
```

List:

```bash
find ssh-keys -type f -maxdepth 1 -print
```

Never copy production private keys into the directory.

---

# 43. SSH Key Passphrase Research

Generate a laboratory key with a passphrase:

```bash
ssh-keygen \
  -t ed25519 \
  -f lab_key_protected
```

Set a laboratory-only passphrase when prompted.

Check:

```bash
ssh-keygen -y -f lab_key_protected
```

Use the resulting key only within your authorized environment.

---

# 44. SSH Key Permissions

Check:

```bash
ls -l ssh-keys/
```

Fix:

```bash
chmod 600 ssh-keys/*
```

Verify:

```bash
stat ssh-keys/*
```

Private keys should not be world-readable.

---

# 45. OpenVPN Laboratory

Create a dedicated VPN test environment.

Verify the configuration:

```bash
ls -lah vpn-lab/
```

Inspect available Crowbar options:

```bash
crowbar --help
```

Use only the OpenVPN backend and authentication parameters supported by your installed version.

Do not test third-party VPN services.

---

# 46. Backend Testing

Check available backend names:

```bash
crowbar --help | grep -i backend
```

If the output differs, simply use:

```bash
crowbar --help
```

and inspect the complete list.

---

# 47. Target Validation

Before running an authentication test:

```bash
ip route
```

Verify the target belongs to your laboratory network.

Then:

```bash
ping -c 2 192.168.56.101
```

Check the service:

```bash
nc -vz 192.168.56.101 3389
```

Only continue if the host is your authorized laboratory target.

---

# 48. Troubleshooting

### Command Not Found

```bash
which crowbar
```

Install:

```bash
sudo apt update
sudo apt install crowbar
```

---

### Backend Not Available

Check:

```bash
crowbar --help
```

Use only backends supported by your installed version.

---

### Connection Refused

Check:

```bash
nc -vz 192.168.56.101 3389
```

Scan:

```bash
nmap -sV -p 3389 192.168.56.101
```

Verify the service is running.

---

### RDP Authentication Fails

Test manually:

```bash
xfreerdp /v:192.168.56.101 /u:labuser
```

Verify:

```text
Username
Password
RDP enabled
Network connectivity
Windows account status
NLA configuration
```

---

### SSH Authentication Fails

Test:

```bash
ssh labuser@192.168.56.102
```

For a key:

```bash
ssh -i ssh-keys/lab_key labuser@192.168.56.102
```

Check:

```bash
sudo journalctl -u ssh -n 50
```

---

### Too Many Failures

Reduce the candidate list:

```bash
head -n 5 passwords.txt > small.txt
```

Use the smaller file first.

---

### Account Locked

Stop the test.

Unlock/reset the dedicated laboratory account and document the lockout behavior.

---

# 49. Practical Progression

```text
Level 1
Install
Help
Backend Discovery
Network Verification
Known Credentials

        ↓

Level 2
RDP
SSH
VNC
OpenVPN

        ↓

Level 3
Password Files
Small Candidate Sets
Custom Targets
Output Logging

        ↓

Level 4
SSH Key Testing
RDP Authentication Research
Service Logs
Response Analysis

        ↓

Level 5
Rate Limits
Account Lockout
Authentication Policies
Detection

        ↓

Level 6
Automation
Repeatable Experiments
Multi-Service Labs
SIEM Correlation
Custom Authentication Labs
```

---

# 50. Quick Reference

```bash
# Help
crowbar --help

# Package information
apt show crowbar

# Check RDP
nc -vz 192.168.56.101 3389

# Check SSH
nc -vz 192.168.56.102 22

# Check VNC
nc -vz 192.168.56.103 5900

# Scan laboratory services
nmap -sV 192.168.56.101

# Create password list
cat > passwords.txt <<'EOF'
test
test123
lab123
research123
EOF

# RDP laboratory password test
crowbar \
  -b rdp \
  -s 192.168.56.101/32 \
  -u labuser \
  -C passwords.txt

# Generate numeric candidates
seq -w 0000 9999 > numbers.txt

# Generate LAB + four digits
for i in $(seq -w 0000 9999); do
    echo "LAB$i"
done > lab-passwords.txt

# Generate SSH laboratory key
ssh-keygen \
  -t ed25519 \
  -f lab_key

# Protect private key
chmod 600 lab_key

# Test SSH key manually
ssh \
  -i lab_key \
  labuser@192.168.56.102

# SSH logs
sudo journalctl -u ssh -f

# Save Crowbar output
crowbar \
  -b rdp \
  -s 192.168.56.101/32 \
  -u labuser \
  -C passwords.txt \
  2>&1 | tee crowbar-rdp.log
```

---

# 51. Final Laboratory Workflow

```text
Build Isolated Target
        ↓
Create Dedicated Test Account
        ↓
Identify Authentication Service
        ↓
Verify Network Connectivity
        ↓
Select Crowbar Backend
        ↓
Read Backend Options
        ↓
Verify Manual Authentication
        ↓
Create Small Candidate Dataset
        ↓
Run Controlled Test
        ↓
Monitor Authentication Logs
        ↓
Analyze Lockout / Rate Limiting
        ↓
Test Detection
        ↓
Record Results
        ↓
Document Findings
```

---

# Scope

Crowbar automates authentication testing against supported services.

Use it only against:

* Your own systems
* Isolated laboratory services
* CTF environments
* Explicitly authorized penetration-testing targets

Do not use these commands against third-party accounts, public RDP/SSH/VNC services, VPN providers, or systems without authorization.
