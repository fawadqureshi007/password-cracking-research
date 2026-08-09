# Hydra — Practical Guide

> Practical THC Hydra reference for authorized authentication-security testing, CTFs, and isolated laboratories.

---

## 1. Install

### Kali Linux

```bash
sudo apt update
sudo apt install hydra
```

Check:

```bash
hydra -h
```

Version:

```bash
hydra -V
```

---

# 2. Basic Syntax

General structure:

```bash
hydra [options] <target> <service>
```

Username + password list:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  ssh
```

Only run this against systems you own or are explicitly authorized to test.

---

# 3. Help

```bash
hydra -h
```

More detailed usage:

```bash
hydra -U ssh
```

The `-U` option displays module-specific usage.

Example:

```bash
hydra -U ftp
```

---

# 4. Available Services

Hydra supports numerous protocols depending on the installed build.

Useful examples include:

```text
ssh
ftp
telnet
smtp
imap
pop3
mysql
postgres
rdp
smb
http
https
```

Check your installation:

```bash
hydra -h
```

Do not assume that every protocol is available or behaves identically across Hydra versions.

---

# 5. Create a Lab Password List

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

# 6. Single Username

Against an SSH service on your own machine:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  ssh
```

The same basic structure applies to other supported modules.

---

# 7. Username List

Create:

```bash
cat > users.txt <<'EOF'
labuser
testuser
research
EOF
```

Run:

```bash
hydra \
  -L users.txt \
  -P passwords.txt \
  127.0.0.1 \
  ssh
```

This tests combinations from both files.

---

# 8. Single Username + Single Password

For a controlled authentication test:

```bash
hydra \
  -l labuser \
  -p lab123 \
  127.0.0.1 \
  ssh
```

This is useful for verifying that Hydra and the selected service module are functioning before using a password list.

---

# 9. Custom Port

If your laboratory SSH service runs on port `2222`:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  -s 2222 \
  127.0.0.1 \
  ssh
```

Check the service first:

```bash
nc -vz 127.0.0.1 2222
```

---

# 10. SSH

Basic:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  ssh
```

Username list:

```bash
hydra \
  -L users.txt \
  -P passwords.txt \
  127.0.0.1 \
  ssh
```

Custom port:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  -s 2222 \
  127.0.0.1 \
  ssh
```

---

# 11. FTP

Check the FTP service:

```bash
nc -vz 127.0.0.1 21
```

Run:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  ftp
```

Multiple users:

```bash
hydra \
  -L users.txt \
  -P passwords.txt \
  127.0.0.1 \
  ftp
```

---

# 12. Telnet

Check:

```bash
nc -vz 127.0.0.1 23
```

Run:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  telnet
```

Use Telnet only in an isolated laboratory.

---

# 13. SMTP

Check module support:

```bash
hydra -U smtp
```

Run against your own mail server:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  smtp
```

SMTP authentication mechanisms vary, so inspect the module options before testing.

---

# 14. IMAP

Check:

```bash
hydra -U imap
```

Run against a laboratory mail server:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  imap
```

---

# 15. POP3

Check:

```bash
hydra -U pop3
```

Run:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  pop3
```

---

# 16. MySQL

Check:

```bash
hydra -U mysql
```

Against a local database:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  mysql
```

Use a database account created specifically for the test.

---

# 17. PostgreSQL

Check:

```bash
hydra -U postgres
```

Run:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  postgres
```

---

# 18. SMB

Check:

```bash
hydra -U smb
```

Against an isolated Windows laboratory system:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  192.168.56.101 \
  smb
```

Only use a private lab network or an explicitly authorized target.

---

# 19. RDP

Check:

```bash
hydra -U rdp
```

Against a Windows test VM:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  192.168.56.101 \
  rdp
```

If your build uses a different RDP module name, use the module shown by:

```bash
hydra -h
```

---

# 20. HTTP Basic Authentication

For a laboratory HTTP service using Basic Authentication:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  http-get \
  /
```

The exact syntax depends on the HTTP module and application.

Check:

```bash
hydra -U http-get
```

---

# 21. HTTPS Basic Authentication

Check:

```bash
hydra -U https-get
```

Then use the appropriate HTTPS module supported by your build against your laboratory service.

Example structure:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  https-get \
  /
```

---

# 22. HTTP POST Forms

Hydra can test some HTTP login forms when the request structure and failure condition are known.

For a laboratory application:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  http-post-form \
  "/login.php:user=^USER^&pass=^PASS^:F=Invalid"
```

The placeholders are:

```text
^USER^
^PASS^
```

The general structure is:

```text
/path:POST-data:failure-condition
```

Check the module documentation:

```bash
hydra -U http-post-form
```

Do not use this against third-party login pages.

---

# 23. HTTPS POST Forms

For a laboratory HTTPS application, inspect:

```bash
hydra -U https-post-form
```

Then use the exact parameters required by your application.

General structure:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  https-post-form \
  "/login:user=^USER^&pass=^PASS^:F=Invalid"
```

Verify the form manually before running Hydra.

---

# 24. Find the Login Request

For your own web application, inspect the login request using browser developer tools.

Record:

```text
HTTP Method
Path
Parameter Names
Successful Response
Failed Response
Failure Message
Cookies
```

Example:

```text
POST /login
username=...
password=...
```

Then construct the corresponding laboratory Hydra module parameters.

---

# 25. HTTP Form Debugging

Before Hydra:

```bash
curl -i http://127.0.0.1/login
```

Submit your known test credentials manually.

Example:

```bash
curl -i \
  -X POST \
  -d 'user=labuser&pass=lab123' \
  http://127.0.0.1/login.php
```

Compare the successful and failed responses.

This helps identify the correct failure condition for the Hydra module.

---

# 26. Threads

Hydra can perform concurrent attempts.

Example:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  -t 4 \
  127.0.0.1 \
  ssh
```

Start low:

```text
-t 1
```

Then:

```text
-t 2
-t 4
```

Increase only in your laboratory.

High concurrency can cause:

```text
Connection failures
Service overload
Account lockouts
Rate limiting
Resource exhaustion
```

---

# 27. Stop on First Valid Credential

Hydra can stop after finding a valid login.

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  -f \
  127.0.0.1 \
  ssh
```

For multiple targets or users, review the exact behavior for your Hydra version.

---

# 28. Verbose Output

Use:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  -v \
  127.0.0.1 \
  ssh
```

More detailed debugging:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  -V \
  127.0.0.1 \
  ssh
```

`-V` can produce a large amount of output.

Use it mainly when debugging a small laboratory wordlist.

---

# 29. Save Output

Save the results:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  127.0.0.1 \
  ssh \
  -o hydra-results.txt
```

Read:

```bash
cat hydra-results.txt
```

Never publish output containing real credentials.

---

# 30. Quiet Output

For cleaner automated output:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  -q \
  127.0.0.1 \
  ssh
```

Check:

```bash
hydra -h
```

because output behavior can vary between versions.

---

# 31. Restore Interrupted Jobs

Hydra can create restore information for interrupted sessions.

List files:

```bash
ls -la
```

Look for the restore file created by the running job.

Hydra's help explains the restore-related options:

```bash
hydra -h | grep -i restore
```

For repeatable research, record the exact command and input files so the experiment can be recreated cleanly.

---

# 32. Generate a Lab Password List

Numeric test set:

```bash
seq -w 0000 9999 > numbers.txt
```

Check:

```bash
wc -l numbers.txt
head numbers.txt
tail numbers.txt
```

Run against your own test service:

```bash
hydra \
  -l labuser \
  -P numbers.txt \
  127.0.0.1 \
  ssh
```

---

# 33. Pattern-Based Candidates

Suppose your laboratory account uses:

```text
LAB + 4 digits
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
hydra \
  -l labuser \
  -P lab-passwords.txt \
  127.0.0.1 \
  ssh
```

This lets you measure the effect of reducing the candidate space.

---

# 34. Multiple Usernames

```bash
hydra \
  -L users.txt \
  -P passwords.txt \
  127.0.0.1 \
  ssh
```

For a controlled test:

```text
users.txt
-----------
labuser1
labuser2
labuser3
```

and:

```text
passwords.txt
-------------
Lab123
Research123
Test123
```

---

# 35. Username + Password Pair Files

Some Hydra workflows support pair-based input.

Check:

```bash
hydra -h
```

Look for options related to:

```text
username/password pairs
```

Use these only when your test dataset represents accounts you created for the laboratory.

---

# 36. Multiple Laboratory Hosts

Create:

```bash
cat > lab-hosts.txt <<'EOF'
192.168.56.101
192.168.56.102
192.168.56.103
EOF
```

Hydra supports different target-input mechanisms depending on the version and invocation.

Check:

```bash
hydra -h
```

Only include isolated laboratory hosts.

---

# 37. IPv6 Laboratory Target

If your service listens on IPv6:

```bash
ss -tulpen6
```

Then use the IPv6 address according to the Hydra syntax supported by your version.

Verify connectivity first:

```bash
nc -6 -vz ::1 22
```

---

# 38. Proxy Testing

Hydra supports proxy-related options in some builds.

Check:

```bash
hydra -h | grep -i proxy
```

Do not use proxies to hide unauthorized activity.

In a laboratory, proxies can be useful for studying:

```text
Request routing
Authentication logging
Network segmentation
Traffic inspection
```

---

# 39. Authentication Logs

While testing SSH on a Linux laboratory server:

```bash
sudo journalctl -u ssh -f
```

On systems using `/var/log/auth.log`:

```bash
sudo tail -f /var/log/auth.log
```

Run Hydra from another terminal.

You can then correlate:

```text
Hydra Attempt
      ↓
Network Connection
      ↓
Authentication
      ↓
Server Log
      ↓
Detection / Alert
```

---

# 40. Rate-Limit Research

Start with one thread:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  -t 1 \
  127.0.0.1 \
  ssh
```

Record:

```text
Attempts
Time
Failures
Successful Login
Server Response
```

Then test:

```bash
-t 2
```

and:

```bash
-t 4
```

Compare the results.

---

# 41. Lockout Research

Create several test accounts.

Use a deliberately incorrect list:

```bash
cat > bad-passwords.txt <<'EOF'
wrong1
wrong2
wrong3
wrong4
wrong5
EOF
```

Run:

```bash
hydra \
  -l labuser \
  -P bad-passwords.txt \
  -t 1 \
  127.0.0.1 \
  ssh
```

Monitor the authentication logs.

Measure:

```text
Attempts Before Lockout
Lockout Duration
Server Response
Log Entry
Alert
Recovery Procedure
```

---

# 42. Detection Testing

A useful blue-team laboratory experiment is:

```text
Hydra
  ↓
Authentication Attempts
  ↓
Server Logs
  ↓
SIEM / Detection Rule
  ↓
Alert
```

Record whether the system detects:

```text
Repeated Failures
Multiple Usernames
Rapid Attempts
Distributed Attempts
Repeated Source
Successful Login After Failures
```

---

# 43. Service Comparison

Build an isolated test environment containing:

```text
SSH
FTP
HTTP
Database
Mail
Windows Authentication
```

Use the same small laboratory credential dataset.

Measure:

```text
Service
Module
Attempts
Rate
Lockout
Delay
Logging
Detection
```

Do not compare services using different test conditions.

---

# 44. HTTP Form Research

Create a simple laboratory login application.

Example:

```text
POST /login
username=labuser
password=lab123
```

Failed response:

```text
Invalid credentials
```

Verify manually:

```bash
curl -i \
  -X POST \
  -d 'username=labuser&password=wrong' \
  http://127.0.0.1/login
```

Then verify the correct password:

```bash
curl -i \
  -X POST \
  -d 'username=labuser&password=lab123' \
  http://127.0.0.1/login
```

Only after confirming the responses should you construct the corresponding Hydra module command.

---

# 45. HTTPS Laboratory Application

Run the same test against your own HTTPS application.

Verify:

```bash
curl -k -i https://127.0.0.1/login
```

Then inspect the module:

```bash
hydra -U https-post-form
```

Use the exact syntax required by your application's request format.

---

# 46. Session / Cookie-Based Login

Some applications require:

```text
Session Cookie
CSRF Token
Dynamic Parameter
Redirect
```

A simple static Hydra form may not be appropriate.

First inspect the request in your laboratory:

```text
Browser
   ↓
GET login page
   ↓
Receive session / token
   ↓
POST credentials
   ↓
Server response
```

If the request changes for every attempt, document the application's authentication flow before selecting a testing method.

---

# 47. Failed vs Successful Response

For HTTP form testing, capture both responses:

```bash
curl -i \
  -X POST \
  -d 'username=labuser&password=wrong' \
  http://127.0.0.1/login
```

and:

```bash
curl -i \
  -X POST \
  -d 'username=labuser&password=lab123' \
  http://127.0.0.1/login
```

Compare:

```text
Status Code
Response Body
Redirect
Cookie
Header
```

The difference is what the testing module needs to recognize success or failure.

---

# 48. Automation

Create:

```bash
nano run-hydra.sh
```

```bash
#!/bin/bash

HOST="$1"
USER="$2"
PASSWORDS="$3"
SERVICE="$4"

echo "[+] Host:    $HOST"
echo "[+] User:    $USER"
echo "[+] Service: $SERVICE"
echo "[+] Starting laboratory test..."

hydra \
    -l "$USER" \
    -P "$PASSWORDS" \
    "$HOST" \
    "$SERVICE"
```

Make executable:

```bash
chmod +x run-hydra.sh
```

Run:

```bash
./run-hydra.sh 127.0.0.1 labuser passwords.txt ssh
```

---

# 49. Logging Experiments

Create:

```bash
mkdir -p experiments
```

Run:

```bash
hydra \
  -l labuser \
  -P passwords.txt \
  -t 1 \
  -v \
  127.0.0.1 \
  ssh \
  2>&1 | tee experiments/ssh-test-01.log
```

Review:

```bash
less experiments/ssh-test-01.log
```

---

# 50. Experiment Matrix

Create a table:

```text
Service | Threads | Users | Passwords | Result | Time
-------------------------------------------------------
SSH     | 1       | 1     | 10        | ...    | ...
SSH     | 2       | 1     | 10        | ...    | ...
SSH     | 4       | 1     | 10        | ...    | ...
FTP     | 1       | 1     | 10        | ...    | ...
HTTP    | 1       | 1     | 10        | ...    | ...
```

Keep the same:

```text
Host
Credential Dataset
Network
Service Configuration
```

when comparing thread counts.

---

# 51. Troubleshooting

### Command Not Found

```bash
which hydra
```

If missing:

```bash
sudo apt update
sudo apt install hydra
```

---

### Service Not Found

Check connectivity:

```bash
nc -vz 127.0.0.1 22
```

Check listening services:

```bash
ss -tulpen
```

---

### Module Error

Inspect module usage:

```bash
hydra -U <module>
```

Check supported modules:

```bash
hydra -h
```

---

### Authentication Always Fails

First authenticate manually.

SSH:

```bash
ssh labuser@127.0.0.1
```

FTP:

```bash
ftp 127.0.0.1
```

HTTP:

```bash
curl -i http://127.0.0.1/login
```

If manual authentication fails, fix the laboratory service before troubleshooting Hydra.

---

### Too Many Connection Errors

Reduce threads:

```bash
-t 1
```

Then:

```bash
-t 2
```

Check the target:

```bash
ss -s
```

and inspect service logs.

---

### Account Locked

Stop the experiment immediately.

Reset/unlock the laboratory account and adjust the test parameters.

---

# 52. Practical Progression

```text
Level 1
Install
Help
Modules
Single Username
Small Password List

        ↓

Level 2
Username Lists
Custom Ports
SSH
FTP
Databases
Mail

        ↓

Level 3
HTTP Authentication
HTTP Forms
HTTPS
Verbose Debugging
Output Logging

        ↓

Level 4
Threads
Rate Limits
Lockouts
Authentication Logs

        ↓

Level 5
Automation
Multiple Laboratory Hosts
Service Comparison
Detection Testing

        ↓

Level 6
Custom Authentication Labs
CSRF / Session Research
SIEM Correlation
Repeatable Experiments
```

---

# 53. Quick Reference

```bash
# Help
hydra -h

# Version
hydra -V

# Module help
hydra -U ssh

# Single username
hydra -l labuser -P passwords.txt 127.0.0.1 ssh

# Username list
hydra -L users.txt -P passwords.txt 127.0.0.1 ssh

# Single password
hydra -l labuser -p lab123 127.0.0.1 ssh

# Custom port
hydra -l labuser -P passwords.txt -s 2222 127.0.0.1 ssh

# Threads
hydra -l labuser -P passwords.txt -t 2 127.0.0.1 ssh

# Stop after finding a valid credential
hydra -l labuser -P passwords.txt -f 127.0.0.1 ssh

# Verbose
hydra -l labuser -P passwords.txt -v 127.0.0.1 ssh

# Very verbose
hydra -l labuser -P passwords.txt -V 127.0.0.1 ssh

# Save output
hydra -l labuser -P passwords.txt 127.0.0.1 ssh -o results.txt

# HTTP GET
hydra -l labuser -P passwords.txt 127.0.0.1 http-get /

# HTTP POST form
hydra -l labuser -P passwords.txt 127.0.0.1 http-post-form \
"/login.php:user=^USER^&pass=^PASS^:F=Invalid"
```

---

# 54. Final Laboratory Workflow

```text
Create Test Service
        ↓
Create Test Accounts
        ↓
Create Small Credential Dataset
        ↓
Verify Manual Authentication
        ↓
Select Hydra Module
        ↓
Run Single-Threaded Test
        ↓
Monitor Logs
        ↓
Increase Candidate Set
        ↓
Test Concurrency
        ↓
Test Lockout / Rate Limiting
        ↓
Test Detection
        ↓
Record Results
        ↓
Document Findings
```

---

# Scope

Hydra performs live authentication attempts against network services.

Use it only against:

* Your own systems
* Isolated laboratory services
* CTF environments
* Explicitly authorized penetration-testing targets

Do not use these commands against third-party accounts, public login portals, or systems without authorization.
