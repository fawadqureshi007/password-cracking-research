# Ncrack — Practical Guide

> Practical Ncrack reference for authorized network-authentication auditing and controlled laboratory research.

---

## 1. Install

### Kali Linux

```bash
sudo apt update
sudo apt install ncrack
```

Check:

```bash
ncrack --version
```

Help:

```bash
ncrack -h
```

---

# 2. Basic Syntax

General structure:

```bash
ncrack [options] [service://target]
```

Example against a laboratory SSH service:

```bash
ncrack \
  -p 22 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1
```

Only use this against systems you own or have explicit authorization to test.

---

# 3. Create a Lab Password List

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

# 4. Create a Username List

```bash
cat > users.txt <<'EOF'
labuser
testuser
research
EOF
```

---

# 5. SSH

Check the service:

```bash
nc -vz 127.0.0.1 22
```

Basic test:

```bash
ncrack \
  -p 22 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1
```

Multiple usernames:

```bash
ncrack \
  -p 22 \
  -U users.txt \
  -P passwords.txt \
  127.0.0.1
```

---

# 6. Custom SSH Port

If your laboratory SSH server listens on `2222`:

```bash
ncrack \
  -p 2222 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1
```

Verify first:

```bash
nc -vz 127.0.0.1 2222
```

---

# 7. FTP

Check:

```bash
nc -vz 127.0.0.1 21
```

Run:

```bash
ncrack \
  -p 21 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1
```

Multiple users:

```bash
ncrack \
  -p 21 \
  -U users.txt \
  -P passwords.txt \
  127.0.0.1
```

---

# 8. RDP

For an isolated Windows laboratory VM:

```bash
ncrack \
  -p 3389 \
  --user labuser \
  -P passwords.txt \
  192.168.56.101
```

Check connectivity:

```bash
nc -vz 192.168.56.101 3389
```

---

# 9. SMB

Check whether your installed version supports the required SMB service.

```bash
ncrack -h
```

Against your isolated Windows test machine:

```bash
ncrack \
  -p 445 \
  --user labuser \
  -P passwords.txt \
  192.168.56.101
```

---

# 10. Telnet

Check:

```bash
nc -vz 127.0.0.1 23
```

Run:

```bash
ncrack \
  -p 23 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1
```

Use Telnet only in an isolated laboratory.

---

# 11. MySQL

Check:

```bash
nc -vz 127.0.0.1 3306
```

Run:

```bash
ncrack \
  -p 3306 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1
```

Use a database account created specifically for your experiment.

---

# 12. PostgreSQL

Check:

```bash
nc -vz 127.0.0.1 5432
```

Run:

```bash
ncrack \
  -p 5432 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1
```

---

# 13. Multiple Ports

For an isolated laboratory host:

```bash
ncrack \
  -p 22,21,3389 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1
```

Only include services you intentionally configured for the experiment.

---

# 14. Multiple Hosts

Ncrack can work with multiple targets.

Example laboratory network:

```text
192.168.56.101
192.168.56.102
192.168.56.103
```

Check the exact target-list syntax supported by your version:

```bash
ncrack -h
```

Keep target lists restricted to your lab.

---

# 15. Username + Password Testing

Single username:

```bash
ncrack \
  -p 22 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1
```

Username file:

```bash
ncrack \
  -p 22 \
  -U users.txt \
  -P passwords.txt \
  127.0.0.1
```

---

# 16. Authentication Testing Workflow

Before running Ncrack:

```bash
ss -tulpen
```

Identify the service.

Then:

```bash
nc -vz 127.0.0.1 22
```

Verify manual authentication:

```bash
ssh labuser@127.0.0.1
```

Then start with a very small password list:

```bash
ncrack \
  -p 22 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1
```

---

# 17. Connection Rate

Ncrack provides timing and connection-control options.

View:

```bash
ncrack -h
```

Look for options related to:

```text
Connection rate
Timing
Parallel connections
Retries
```

Start conservatively in a laboratory.

The objective is to measure authentication behavior rather than overload the service.

---

# 18. Timing Experiments

Run a small test:

```bash
ncrack \
  -p 22 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1
```

Record:

```text
Start time
End time
Number of candidates
Successful attempts
Failed attempts
Connection failures
```

Repeat after changing only one timing parameter.

---

# 19. Output

Save the terminal output:

```bash
ncrack \
  -p 22 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1 \
  2>&1 | tee ncrack-test.log
```

Review:

```bash
less ncrack-test.log
```

Keep credentials out of public repositories.

---

# 20. Verbose Output

Check the available verbosity options:

```bash
ncrack -h
```

Use the supported verbosity flag with a small laboratory test.

Example:

```bash
ncrack \
  -v \
  -p 22 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1
```

---

# 21. Stop Conditions

Inspect the help for stopping behavior:

```bash
ncrack -h
```

For repeatable research, record:

```text
Target
Service
Username Dataset
Password Dataset
Timing
Threads / Connections
Ncrack Version
Result
```

---

# 22. Password Candidate Generation

Create a numeric test set:

```bash
seq -w 0000 9999 > numbers.txt
```

Check:

```bash
wc -l numbers.txt
```

Use against your lab:

```bash
ncrack \
  -p 22 \
  --user labuser \
  -P numbers.txt \
  127.0.0.1
```

---

# 23. Pattern-Based Passwords

Suppose your laboratory account uses:

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

Run:

```bash
ncrack \
  -p 22 \
  --user labuser \
  -P lab-passwords.txt \
  127.0.0.1
```

This is useful for measuring how predictable password patterns reduce the search space.

---

# 24. Username Generation

Create a controlled username dataset:

```bash
cat > users.txt <<'EOF'
labuser1
labuser2
labuser3
testuser
research
EOF
```

Run:

```bash
ncrack \
  -p 22 \
  -U users.txt \
  -P passwords.txt \
  127.0.0.1
```

---

# 25. Rate-Limit Research

Start with a small list:

```bash
cat > rate-test.txt <<'EOF'
wrong1
wrong2
wrong3
wrong4
wrong5
EOF
```

Run:

```bash
ncrack \
  -p 22 \
  --user labuser \
  -P rate-test.txt \
  127.0.0.1
```

Record:

```text
Attempts
Time
Response Delay
Connection Errors
Lockout
```

Repeat using different timing settings supported by your version.

---

# 26. Lockout Testing

Create a dedicated test account.

Use intentionally incorrect passwords:

```bash
cat > lockout-test.txt <<'EOF'
incorrect1
incorrect2
incorrect3
incorrect4
incorrect5
EOF
```

Run:

```bash
ncrack \
  -p 22 \
  --user labuser \
  -P lockout-test.txt \
  127.0.0.1
```

Monitor the server.

Record:

```text
Attempts before lockout
Lockout duration
Authentication response
Server logs
Alert generated
```

Never perform lockout testing against accounts you do not control.

---

# 27. SSH Authentication Logs

On a Linux laboratory server:

```bash
sudo journalctl -u ssh -f
```

On systems using `auth.log`:

```bash
sudo tail -f /var/log/auth.log
```

Run Ncrack from another terminal.

Observe:

```text
Ncrack
  ↓
TCP Connection
  ↓
SSH Authentication
  ↓
Server
  ↓
Authentication Log
```

---

# 28. FTP Logs

Locate the FTP service log:

```bash
sudo journalctl -u vsftpd -f
```

The exact service name depends on your FTP implementation.

Monitor the logs while running the laboratory test.

---

# 29. Windows Laboratory Monitoring

For a Windows test machine, monitor authentication events using:

```text
Event Viewer
    ↓
Windows Logs
    ↓
Security
```

Look for failed and successful authentication events.

Compare the Windows event timestamps with the Ncrack test.

---

# 30. Detection Research

Create an experiment:

```text
Ncrack
   ↓
Authentication Attempts
   ↓
Service Logs
   ↓
Detection Rule
   ↓
Alert
```

Measure whether your security controls detect:

```text
Repeated failures
Rapid failures
Multiple usernames
Multiple services
Repeated source IP
Successful login after failures
```

---

# 31. Service Comparison

Create several isolated services:

```text
SSH
FTP
RDP
SMB
Database
```

Use the same controlled credential dataset.

Record:

```text
Service
Port
Attempts
Response Time
Lockout
Rate Limiting
Logging
Detection
```

This gives you a repeatable authentication-security experiment.

---

# 32. Multiple Laboratory Hosts

Create a private test network:

```text
Attacker VM
     |
     +---- Target 1
     |
     +---- Target 2
     |
     +---- Target 3
```

Example:

```text
192.168.56.101
192.168.56.102
192.168.56.103
```

Verify connectivity:

```bash
for ip in 192.168.56.101 192.168.56.102 192.168.56.103; do
    ping -c 1 "$ip"
done
```

Then use Ncrack's supported target syntax:

```bash
ncrack -h
```

---

# 33. Lab Automation

Create:

```bash
nano run-ncrack.sh
```

```bash
#!/bin/bash

HOST="$1"
PORT="$2"
USER="$3"
PASSWORDS="$4"

echo "[+] Target: $HOST"
echo "[+] Port:   $PORT"
echo "[+] User:   $USER"

ncrack \
    -p "$PORT" \
    --user "$USER" \
    -P "$PASSWORDS" \
    "$HOST"
```

Make executable:

```bash
chmod +x run-ncrack.sh
```

Run:

```bash
./run-ncrack.sh 127.0.0.1 22 labuser passwords.txt
```

---

# 34. Experiment Directory

Create:

```bash
mkdir -p experiments
```

Run:

```bash
ncrack \
  -p 22 \
  --user labuser \
  -P passwords.txt \
  127.0.0.1 \
  2>&1 | tee experiments/ssh-test-01.log
```

---

# 35. Experiment Naming

Use consistent names:

```text
experiments/
├── ssh-test-01.log
├── ssh-test-02.log
├── ftp-test-01.log
├── rdp-test-01.log
└── comparison.md
```

Record the exact command used for each experiment.

---

# 36. Benchmarking

For each experiment record:

```text
Ncrack Version:
Target:
Service:
Port:
Username:
Password List:
Number of Candidates:
Timing:
Start:
End:
Result:
```

Example:

```text
Ncrack Version: 0.x
Target: 127.0.0.1
Service: SSH
Port: 22
Username: labuser
Candidates: 100
Timing: controlled
Result: documented
```

---

# 37. Authentication Defense Experiment

Configure your laboratory SSH server with authentication protections.

Then test:

```text
Baseline
   ↓
Ncrack
   ↓
Measure failures
   ↓
Enable rate limiting
   ↓
Ncrack
   ↓
Compare results
```

Record:

```text
Before protection
After protection
Attempts
Response delay
Lockout
Alerts
```

---

# 38. Password Policy Experiment

Create laboratory accounts using different password patterns.

Example categories:

```text
Short password
Dictionary password
Predictable password
Long random password
Passphrase
```

Use controlled test data.

Measure:

```text
Candidate-space size
Time to test
Authentication rate
Successful discovery
```

Do not use real user passwords for experiments.

---

# 39. Troubleshooting

### Ncrack Not Found

```bash
which ncrack
```

Install:

```bash
sudo apt update
sudo apt install ncrack
```

---

### Connection Refused

Check:

```bash
ss -tulpen
```

Test:

```bash
nc -vz 127.0.0.1 22
```

---

### Authentication Fails

Test manually:

```bash
ssh labuser@127.0.0.1
```

Verify your password list:

```bash
cat passwords.txt
```

---

### Module / Service Error

Check:

```bash
ncrack -h
```

Confirm that the target protocol and port are supported by the installed version.

---

### Too Many Connection Errors

Reduce the test rate using the timing/concurrency controls supported by your version.

Check:

```bash
ncrack -h
```

Then start with the most conservative setting.

---

### Account Locked

Stop the test.

Unlock/reset the dedicated laboratory account before continuing.

---

# 40. Practical Workflow

```text
Create Lab Service
        ↓
Create Test Account
        ↓
Verify Manual Login
        ↓
Create Small Password List
        ↓
Identify Service / Port
        ↓
Run Ncrack
        ↓
Monitor Authentication Logs
        ↓
Record Result
        ↓
Test Rate Limiting
        ↓
Test Lockout
        ↓
Test Detection
        ↓
Compare Configurations
        ↓
Document Findings
```

---

# 41. Ncrack vs Hydra vs Medusa

| Tool            | Main Use                               |
| --------------- | -------------------------------------- |
| Ncrack          | Network authentication auditing        |
| Hydra           | Broad protocol authentication testing  |
| Medusa          | Parallelized network login testing     |
| John the Ripper | Offline password/hash auditing         |
| Hashcat         | High-performance offline hash recovery |

Ncrack belongs in the **live authentication** portion of this project.

John and Hashcat belong in the **offline password recovery** portion.

---

# 42. Quick Reference

```bash
# Help
ncrack -h

# Version
ncrack --version

# SSH
ncrack -p 22 --user labuser -P passwords.txt 127.0.0.1

# SSH custom port
ncrack -p 2222 --user labuser -P passwords.txt 127.0.0.1

# Username list
ncrack -p 22 -U users.txt -P passwords.txt 127.0.0.1

# FTP
ncrack -p 21 --user labuser -P passwords.txt 127.0.0.1

# RDP
ncrack -p 3389 --user labuser -P passwords.txt 192.168.56.101

# SMB
ncrack -p 445 --user labuser -P passwords.txt 192.168.56.101

# MySQL
ncrack -p 3306 --user labuser -P passwords.txt 127.0.0.1

# PostgreSQL
ncrack -p 5432 --user labuser -P passwords.txt 127.0.0.1

# Save output
ncrack -p 22 --user labuser -P passwords.txt 127.0.0.1 \
  2>&1 | tee ncrack-test.log
```

---

# 43. Practical Progression

```text
LEVEL 1
Install
Help
Single User
Small Wordlist
SSH

        ↓

LEVEL 2
Username Lists
FTP
RDP
SMB
Database Services
Custom Ports

        ↓

LEVEL 3
Timing
Concurrency
Logging
Output Collection
Candidate Generation

        ↓

LEVEL 4
Rate-Limit Testing
Lockout Testing
Authentication Monitoring
Detection Testing

        ↓

LEVEL 5
Automation
Multiple Lab Hosts
Service Comparison
Repeatable Benchmarks

        ↓

LEVEL 6
Authentication Defense Research
SIEM Correlation
Password Policy Experiments
Controlled Red-Team Simulations
```

---

# 44. Final Checklist

```text
[ ] Target is authorized
[ ] Service identified
[ ] Port verified
[ ] Manual authentication tested
[ ] Dedicated test account created
[ ] Synthetic password list created
[ ] Conservative test started
[ ] Logs monitored
[ ] Lockout policy understood
[ ] Results recorded
[ ] Credentials removed from public output
```

---

## Scope

Ncrack performs live authentication attempts against network services.

Use it only against:

* Systems you own
* Isolated laboratory machines
* CTF environments
* Explicitly authorized penetration-testing targets

Do not use these commands against third-party accounts or public services without authorization.
