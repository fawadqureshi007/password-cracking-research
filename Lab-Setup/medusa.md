# Medusa — Practical Guide

> Practical Medusa reference for authorized password-auditing labs, CTFs, and self-owned services.

---

## 1. Install

### Kali Linux

```bash
sudo apt update
sudo apt install medusa
```

Check:

```bash
medusa -h
```

Version:

```bash
medusa -V
```

---

## 2. Basic Syntax

The general structure is:

```bash
medusa -h <host> -u <username> -P <password-list> -M <module>
```

Example against a local lab service:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M ssh
```

Only run this against a service you control or have explicit authorization to test.

---

# 3. Help

Full help:

```bash
medusa -h
```

Module help:

```bash
medusa -d
```

List available modules:

```bash
medusa -d
```

---

# 4. Modules

Medusa uses modules for different authentication services.

List them:

```bash
medusa -d
```

Common modules may include:

```text
ssh
ftp
http
imap
pop3
smtp
mysql
postgres
telnet
rdp
smbnt
```

The exact list depends on the installed Medusa build.

Check a module:

```bash
medusa -M ssh -q
```

For another module:

```bash
medusa -M ftp -q
```

---

# 5. Local Lab Target

For safe testing, create a service on your own machine or isolated VM.

Check listening services:

```bash
ss -tulpen
```

Example:

```text
127.0.0.1:22
```

Test connectivity:

```bash
nc -vz 127.0.0.1 22
```

Then perform the Medusa test against:

```text
127.0.0.1
```

---

# 6. Single Username

Create:

```bash
nano passwords.txt
```

Example:

```text
test123
password
Password123
lab123
```

Run:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M ssh
```

---

# 7. Username File

Create:

```bash
nano users.txt
```

Example:

```text
labuser
testuser
research
```

Run:

```bash
medusa \
  -h 127.0.0.1 \
  -U users.txt \
  -P passwords.txt \
  -M ssh
```

This tests the combinations from the supplied username and password sets.

---

# 8. Password File

Specify:

```bash
-P passwords.txt
```

Example:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M ssh
```

---

# 9. Multiple Hosts — Lab Only

For an isolated lab network, create:

```bash
nano hosts.txt
```

Example:

```text
192.168.56.101
192.168.56.102
192.168.56.103
```

Use the host-file option supported by your installed version.

Check:

```bash
medusa -h
```

Do not use host lists containing systems you do not own or have permission to test.

---

# 10. SSH

Check the SSH module:

```bash
medusa -M ssh -q
```

Single user:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M ssh
```

Username list:

```bash
medusa \
  -h 127.0.0.1 \
  -U users.txt \
  -P passwords.txt \
  -M ssh
```

---

# 11. FTP

Check:

```bash
medusa -M ftp -q
```

Run against your laboratory FTP server:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M ftp
```

---

# 12. HTTP Authentication

Check available HTTP modules:

```bash
medusa -d | grep -i http
```

Inspect the selected module:

```bash
medusa -M <http-module> -q
```

Then follow the module's required options:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M <http-module>
```

HTTP authentication schemes differ, so do not assume that every login page is supported by the same Medusa module.

---

# 13. MySQL

Check:

```bash
medusa -M mysql -q
```

Against a local MySQL laboratory instance:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M mysql
```

Use a database account created specifically for testing.

---

# 14. PostgreSQL

Check:

```bash
medusa -M postgres -q
```

Run:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M postgres
```

---

# 15. SMB

Check:

```bash
medusa -M smbnt -q
```

Against an isolated Windows laboratory machine:

```bash
medusa \
  -h 192.168.56.101 \
  -u labuser \
  -P passwords.txt \
  -M smbnt
```

The target must be part of your authorized lab.

---

# 16. Telnet

Check:

```bash
medusa -M telnet -q
```

Run:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M telnet
```

Telnet should only be used in an isolated laboratory because it is insecure by design.

---

# 17. RDP

Check whether the installed build contains an RDP module:

```bash
medusa -d | grep -i rdp
```

If available:

```bash
medusa \
  -h 192.168.56.101 \
  -u labuser \
  -P passwords.txt \
  -M <rdp-module>
```

Use only an isolated Windows test machine.

---

# 18. IMAP

Check:

```bash
medusa -d | grep -i imap
```

Inspect:

```bash
medusa -M <imap-module> -q
```

Run against your own mail server:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M <imap-module>
```

---

# 19. POP3

Check:

```bash
medusa -d | grep -i pop
```

Inspect:

```bash
medusa -M <pop3-module> -q
```

Then:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M <pop3-module>
```

---

# 20. SMTP

Check:

```bash
medusa -d | grep -i smtp
```

Inspect:

```bash
medusa -M smtp -q
```

Use only against a mail server in your laboratory.

---

# 21. Username + Password Pair Testing

Create:

```bash
nano users.txt
nano passwords.txt
```

Example:

```text
labuser
testuser
```

and:

```text
lab123
test123
Password123
```

Run:

```bash
medusa \
  -h 127.0.0.1 \
  -U users.txt \
  -P passwords.txt \
  -M ssh
```

---

# 22. Port Selection

If your laboratory service is running on a non-standard port, specify the port supported by the selected module.

Check:

```bash
medusa -h
```

Example:

```bash
medusa \
  -h 127.0.0.1 \
  -n 2222 \
  -u labuser \
  -P passwords.txt \
  -M ssh
```

---

# 23. Module Options

Modules may have their own parameters.

Inspect:

```bash
medusa -M ssh -q
```

or:

```bash
medusa -M ftp -q
```

Use the output to determine the options required by that module.

Do not copy options between modules without checking their module documentation.

---

# 24. Verbose Output

Increase verbosity:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M ssh \
  -v 6
```

If the output becomes excessive, lower the verbosity.

Check the supported verbosity levels:

```bash
medusa -h
```

---

# 25. Debugging

For laboratory troubleshooting:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M ssh \
  -v 6
```

Check the service independently:

```bash
nc -vz 127.0.0.1 22
```

Then test normal authentication manually before blaming Medusa.

---

# 26. Threading

Medusa supports concurrent login attempts.

Check the installed help:

```bash
medusa -h
```

Then configure the thread option supported by your version.

Example:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M ssh \
  -t 4
```

Start with a low value in a laboratory.

Higher concurrency can cause:

```text
Connection failures
Service instability
Rate limiting
Resource exhaustion
```

---

# 27. Connection Testing

Before launching a password audit:

```bash
ping -c 1 127.0.0.1
```

Check the service:

```bash
nc -vz 127.0.0.1 22
```

For SSH:

```bash
ssh labuser@127.0.0.1
```

For FTP:

```bash
ftp 127.0.0.1
```

This confirms that the service itself is reachable.

---

# 28. Build a Lab Wordlist

Create:

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

Run:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M ssh
```

---

# 29. Generate Candidates

Generate synthetic numeric passwords:

```bash
seq -w 0000 9999 > numbers.txt
```

Check:

```bash
head numbers.txt
tail numbers.txt
wc -l numbers.txt
```

Use against your isolated test service:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P numbers.txt \
  -M ssh
```

This is useful for studying how a predictable password space affects an authentication service.

---

# 30. Targeted Candidate Lists

Suppose your laboratory password follows:

```text
LAB + 4 digits
```

Generate:

```bash
for i in $(seq -w 0000 9999); do
    echo "LAB$i"
done > lab-passwords.txt
```

Run:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P lab-passwords.txt \
  -M ssh
```

This is much more controlled than using an unnecessarily large wordlist.

---

# 31. Password Spray Research

For a controlled lab containing multiple test accounts, you can study how a single candidate behaves across accounts.

Create:

```bash
nano users.txt
```

Example:

```text
labuser1
labuser2
labuser3
```

Create:

```bash
nano spray.txt
```

Example:

```text
LabPassword2026!
```

Then test against the isolated service according to the module's supported username/password behavior.

The purpose of the experiment is to measure account-lockout and authentication-defense behavior in your own environment.

---

# 32. Authentication Defense Testing

A useful lab experiment is to configure:

```text
Multiple Test Accounts
        +
Known Passwords
        +
Authentication Logging
```

Then observe:

```text
Failed Attempts
Successful Attempts
Lockout Threshold
Delay
Rate Limiting
Logging
Alerting
```

Medusa becomes the controlled authentication-attempt generator.

---

# 33. Rate-Limit Testing

Start conservatively:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M ssh \
  -t 1
```

Then increase concurrency only inside your laboratory:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M ssh \
  -t 2
```

Compare the service behavior.

Record:

```text
Threads
Attempts
Successful Attempts
Rejected Attempts
Connection Errors
Service CPU
Service Memory
```

---

# 34. Logs

While testing a Linux SSH laboratory server:

```bash
sudo journalctl -u ssh
```

Depending on the distribution:

```bash
sudo tail -f /var/log/auth.log
```

For another service, monitor its corresponding authentication log.

This lets you correlate:

```text
Medusa Attempt
      ↓
Network Request
      ↓
Authentication Event
      ↓
Server Log
      ↓
Detection
```

---

# 35. Detecting Lockouts

Create several controlled test accounts.

Run a small password list:

```bash
medusa \
  -h 127.0.0.1 \
  -U users.txt \
  -P passwords.txt \
  -M ssh \
  -t 1
```

Monitor:

```text
Authentication Logs
Account Status
Lockout Events
Service Response
```

Never perform lockout testing against accounts you do not control.

---

# 36. Comparing Services

Use the same synthetic password list against separate laboratory services:

```text
SSH
FTP
Database
Mail
SMB
```

Record:

```text
Service
Module
Attempts
Success
Failure
Rate Limit
Lockout
Logging
```

This creates a useful authentication-security comparison.

---

# 37. Benchmarking

Medusa is not a hash-cracking benchmark in the same sense as John or Hashcat.

For live authentication research, measure:

```text
Attempts/second
Successful Attempts
Failed Attempts
Connection Errors
Latency
CPU Usage
Memory Usage
```

Keep the test environment constant.

---

# 38. Automation

Create a basic laboratory wrapper:

```bash
#!/bin/bash

HOST="$1"
USER="$2"
PASSWORDS="$3"
MODULE="$4"

echo "[+] Target: $HOST"
echo "[+] User:   $USER"
echo "[+] Module: $MODULE"

medusa \
    -h "$HOST" \
    -u "$USER" \
    -P "$PASSWORDS" \
    -M "$MODULE"
```

Save:

```bash
nano run-medusa.sh
```

Make executable:

```bash
chmod +x run-medusa.sh
```

Run:

```bash
./run-medusa.sh 127.0.0.1 labuser passwords.txt ssh
```

---

# 39. Experiment Logging

Create a directory:

```bash
mkdir -p experiments
```

Run and save output:

```bash
medusa \
  -h 127.0.0.1 \
  -u labuser \
  -P passwords.txt \
  -M ssh \
  -v 4 \
  2>&1 | tee experiments/ssh-test-01.log
```

Review:

```bash
less experiments/ssh-test-01.log
```

---

# 40. Test Matrix

Create a simple research matrix:

```text
Service     Users    Passwords    Threads    Result
----------------------------------------------------
SSH         1        10           1          ...
SSH         1        10           4          ...
FTP         1        10           1          ...
MySQL       1        10           1          ...
```

This allows you to compare authentication behavior.

---

# 41. Troubleshooting

### Module Not Found

Check:

```bash
medusa -d
```

Search:

```bash
medusa -d | grep -i ssh
```

If the required module is missing, install the appropriate Medusa package/build.

---

### Connection Refused

Check:

```bash
ss -tulpen
```

Test:

```bash
nc -vz 127.0.0.1 <port>
```

Check the service:

```bash
sudo systemctl status <service>
```

---

### Authentication Always Fails

Verify manually first.

For SSH:

```bash
ssh labuser@127.0.0.1
```

Then check:

```bash
cat passwords.txt
```

Make sure the expected laboratory password actually exists in the candidate list.

---

### Account Locked

Stop the test.

Check the laboratory authentication policy and unlock/reset the test account before continuing.

---

### Too Many Connection Errors

Reduce concurrency:

```bash
-t 1
```

Then increase gradually.

Also check:

```bash
ss -s
```

and the target service logs.

---

# 42. Practical Workflow

Use this sequence for a new authorized laboratory target:

```text
1. Identify Service
       ↓
2. Confirm Authorization
       ↓
3. Confirm Connectivity
       ↓
4. Identify Medusa Module
       ↓
5. Test Manual Login
       ↓
6. Create Small Password List
       ↓
7. Run Single-Threaded Test
       ↓
8. Monitor Logs
       ↓
9. Increase Test Scope
       ↓
10. Measure Results
       ↓
11. Test Authentication Defenses
       ↓
12. Document Findings
```

---

# 43. Quick Reference

```bash
# Help
medusa -h

# Version
medusa -V

# Modules
medusa -d

# Module information
medusa -M ssh -q

# Single username
medusa -h 127.0.0.1 -u labuser -P passwords.txt -M ssh

# Multiple usernames
medusa -h 127.0.0.1 -U users.txt -P passwords.txt -M ssh

# Custom port
medusa -h 127.0.0.1 -n 2222 -u labuser -P passwords.txt -M ssh

# Verbose
medusa -h 127.0.0.1 -u labuser -P passwords.txt -M ssh -v 6

# Controlled concurrency
medusa -h 127.0.0.1 -u labuser -P passwords.txt -M ssh -t 2

# Save output
medusa -h 127.0.0.1 -u labuser -P passwords.txt -M ssh 2>&1 | tee test.log
```

---

# 44. Practical Lab Progression

```text
Level 1
Installation
Modules
Single Username
Small Password List

        ↓

Level 2
Username Lists
Service Modules
Custom Ports
Verbose Output
Sessions / Logging

        ↓

Level 3
Multiple Accounts
Candidate Generation
Concurrency
Authentication Logs

        ↓

Level 4
Rate-Limit Testing
Lockout Testing
Detection Testing
Service Comparison

        ↓

Level 5
Automation
Experiment Matrices
Performance Measurement
Authentication Defense Research
```

---

# 45. Final Lab Checklist

```text
[ ] Target belongs to my lab / authorized scope
[ ] Service identified
[ ] Module identified
[ ] Manual authentication tested
[ ] Small candidate list prepared
[ ] Concurrency kept controlled
[ ] Authentication logs monitored
[ ] Lockout policy understood
[ ] Results recorded
[ ] Sensitive data removed
```

---

## Scope

Medusa generates authentication attempts against network services. Use it only against systems where you have explicit permission to test.

For offline password recovery, use the dedicated John the Ripper and Hashcat sections of this research project.
