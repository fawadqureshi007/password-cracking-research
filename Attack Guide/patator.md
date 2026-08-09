# Patator — Practical Guide

> Practical Patator reference for authorized authentication-security testing, CTFs, and isolated laboratories.

---

## 1. Install

### Kali Linux

```bash
sudo apt update
sudo apt install patator
```

Check:

```bash
patator --help
```

If your installation exposes the Python entry point:

```bash
patator.py --help
```

Locate it:

```bash
which patator
```

---

# 2. Basic Syntax

General structure:

```bash
patator <module> [options]
```

Example:

```bash
patator ssh_login --help
```

Unlike Hydra, Patator module parameters use:

```text
option=value
```

rather than Hydra-style positional service arguments.

---

# 3. Help

General help:

```bash
patator --help
```

Module-specific help:

```bash
patator ssh_login --help
```

FTP:

```bash
patator ftp_login --help
```

HTTP:

```bash
patator http_fuzz --help
```

SMTP:

```bash
patator smtp_login --help
```

---

# 4. Available Modules

Check your installed version:

```bash
patator --help
```

Common modules include:

```text
ftp_login
ssh_login
telnet_login
smtp_login
smtp_vrfy
smtp_rcpt
http_fuzz
pop_login
imap_login
ldap_login
dcom_login
smb_login
rlogin_login
mssql_login
oracle_login
mysql_login
pgsql_login
vnc_login
snmp_login
unzip_pass
dns_forward
dns_reverse
```

Additional modules can exist depending on the Patator package/version.

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

# 6. Create a Username List

```bash
cat > users.txt <<'EOF'
labuser
testuser
research
operator
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

# 7. Patator FILE Syntax

Patator uses numbered `FILE` placeholders.

Basic example:

```bash
patator ssh_login \
  host=127.0.0.1 \
  user=FILE0 \
  password=FILE1 \
  0=users.txt \
  1=passwords.txt
```

The first `FILE0` uses:

```text
0=users.txt
```

and `FILE1` uses:

```text
1=passwords.txt
```

Patator generates the combinations from the supplied payload sets.

---

# 8. Single Username + Password List

For an SSH service in your own laboratory:

```bash
patator ssh_login \
  host=127.0.0.1 \
  user=labuser \
  password=FILE0 \
  0=passwords.txt
```

This tests the password list against one laboratory account.

---

# 9. Username List + Password List

```bash
patator ssh_login \
  host=127.0.0.1 \
  user=FILE0 \
  password=FILE1 \
  0=users.txt \
  1=passwords.txt
```

This creates the Cartesian product of the two lists by default.

---

# 10. Single Mode

Test the username itself as the password:

```bash
patator ssh_login \
  host=127.0.0.1 \
  user=FILE0 \
  password=FILE0 \
  0=users.txt
```

This is useful for controlled testing of weak `username=password` policies.

---

# 11. SSH

Check the module:

```bash
patator ssh_login --help
```

Basic laboratory test:

```bash
patator ssh_login \
  host=127.0.0.1 \
  user=labuser \
  password=FILE0 \
  0=passwords.txt
```

Username list:

```bash
patator ssh_login \
  host=127.0.0.1 \
  user=FILE0 \
  password=FILE1 \
  0=users.txt \
  1=passwords.txt
```

The documented SSH module supports `host`, `user`, and `password` fuzzing.

---

# 12. SSH Custom Port

Check the module first:

```bash
patator ssh_login --help
```

If your laboratory SSH service listens on `2222`, configure the module's `port` option according to the installed module help.

Example structure:

```bash
patator ssh_login \
  host=127.0.0.1 \
  port=2222 \
  user=labuser \
  password=FILE0 \
  0=passwords.txt
```

Verify the service:

```bash
nc -vz 127.0.0.1 2222
```

---

# 13. SSH Multiple Hosts

Create:

```bash
cat > hosts.txt <<'EOF'
192.168.56.101
192.168.56.102
EOF
```

Then:

```bash
patator ssh_login \
  host=FILE0 \
  user=FILE1 \
  password=FILE2 \
  0=hosts.txt \
  1=users.txt \
  2=passwords.txt
```

Patator can fuzz multiple module parameters simultaneously using numbered payload files.

---

# 14. Stop Testing a Host After Success

Patator's `free` action can stop further testing after a successful condition.

Example:

```bash
patator ssh_login \
  host=FILE0 \
  user=FILE1 \
  password=FILE2 \
  0=hosts.txt \
  1=users.txt \
  2=passwords.txt \
  -x free=host:code=0
```

The documented example uses this pattern to stop testing a host after a valid credential is found.

---

# 15. Stop Testing a User After Success

```bash
patator ssh_login \
  host=FILE0 \
  user=FILE1 \
  password=FILE2 \
  0=hosts.txt \
  1=users.txt \
  2=passwords.txt \
  -x free=host+user:code=0
```

This is useful when testing multiple accounts in an authorized laboratory.

---

# 16. FTP

Check:

```bash
patator ftp_login --help
```

Basic:

```bash
patator ftp_login \
  host=127.0.0.1 \
  user=labuser \
  password=FILE0 \
  0=passwords.txt
```

Username + password lists:

```bash
patator ftp_login \
  host=127.0.0.1 \
  user=FILE0 \
  password=FILE1 \
  0=users.txt \
  1=passwords.txt
```

The official usage documentation demonstrates the same `user=FILE0 password=FILE1` model for FTP.

---

# 17. FTP Failure Filtering

FTP servers commonly return a recognizable failed-login message.

Inspect:

```bash
patator ftp_login --help
```

The documented example uses:

```bash
-x ignore:mesg='Login incorrect.'
```

Example:

```bash
patator ftp_login \
  host=127.0.0.1 \
  user=FILE0 \
  password=FILE1 \
  0=users.txt \
  1=passwords.txt \
  -x ignore:mesg='Login incorrect.'
```

This prevents expected authentication failures from cluttering the result output.

---

# 18. Telnet

Check:

```bash
patator telnet_login --help
```

A Telnet login sequence is interactive, so the module uses input and prompt matching.

Example laboratory structure:

```bash
patator telnet_login \
  host=127.0.0.1 \
  inputs='FILE0\nFILE1' \
  0=users.txt \
  1=passwords.txt \
  prompt_re='login:|Password:'
```

The exact prompts depend on the Telnet server.

Use Telnet only inside an isolated laboratory.

---

# 19. SMTP

Check:

```bash
patator smtp_login --help
```

Basic structure:

```bash
patator smtp_login \
  host=127.0.0.1 \
  user=FILE0 \
  password=FILE1 \
  0=users.txt \
  1=passwords.txt
```

For a laboratory SMTP server, configure the appropriate `helo` value when required.

Example:

```bash
patator smtp_login \
  host=127.0.0.1 \
  helo='ehlo lab.local' \
  user=FILE0 \
  password=FILE1 \
  0=users.txt \
  1=passwords.txt
```

Patator's documentation demonstrates SMTP authentication using the same fuzzable username/password model.

---

# 20. SMTP User Enumeration

Check:

```bash
patator smtp_vrfy --help
```

Laboratory example:

```bash
patator smtp_vrfy \
  host=127.0.0.1 \
  user=FILE0 \
  0=users.txt
```

The module can test usernames through the SMTP `VRFY` mechanism when the server supports it.

---

# 21. SMTP RCPT Testing

Check:

```bash
patator smtp_rcpt --help
```

Example laboratory structure:

```bash
patator smtp_rcpt \
  host=127.0.0.1 \
  user=FILE0@localhost \
  0=users.txt \
  helo='ehlo lab.local' \
  mail_from=root
```

This can be used to study whether the SMTP service reveals recipient validity.

---

# 22. HTTP

Patator uses:

```text
http_fuzz
```

for HTTP/HTTPS testing.

Check:

```bash
patator http_fuzz --help
```

Basic request:

```bash
patator http_fuzz \
  url=http://127.0.0.1/FILE0 \
  0=words.txt
```

This can be used against a laboratory web application for controlled request fuzzing.

---

# 23. HTTP Status Filtering

Example:

```bash
patator http_fuzz \
  url=http://127.0.0.1/FILE0 \
  0=words.txt \
  -x ignore:code=404
```

This ignores normal `404` responses.

Retry server errors:

```bash
patator http_fuzz \
  url=http://127.0.0.1/FILE0 \
  0=words.txt \
  -x ignore,retry:code=500
```

Patator supports action/condition expressions through `-x`.

---

# 24. HTTP GET Login Testing

For a laboratory application using a GET login request:

```bash
patator http_fuzz \
  url='http://127.0.0.1/login?username=labuser&password=FILE0' \
  0=passwords.txt
```

If successful and failed responses differ, use a condition to distinguish them.

Example:

```bash
patator http_fuzz \
  url='http://127.0.0.1/login?username=labuser&password=FILE0' \
  0=passwords.txt \
  -x ignore:fgrep='Invalid credentials'
```

Verify the actual failure text from your laboratory application first.

---

# 25. HTTP POST Login Testing

Check:

```bash
patator http_fuzz --help
```

Example laboratory form:

```text
POST /login
username=labuser
password=FILE0
```

Command:

```bash
patator http_fuzz \
  url=http://127.0.0.1/login \
  method=POST \
  body='username=labuser&password=FILE0' \
  0=passwords.txt
```

Add an application-specific failure condition:

```bash
patator http_fuzz \
  url=http://127.0.0.1/login \
  method=POST \
  body='username=labuser&password=FILE0' \
  0=passwords.txt \
  -x ignore:fgrep='Invalid credentials'
```

The documented Patator HTTP module supports POST bodies and response filtering.

---

# 26. HTTP Cookies

Some applications require a session cookie.

Example:

```bash
patator http_fuzz \
  url=http://127.0.0.1/login \
  header='Cookie: SESSION=LABSESSION' \
  method=POST \
  body='username=labuser&password=FILE0' \
  0=passwords.txt
```

Use only a disposable laboratory session.

---

# 27. HTTP Redirects

Follow redirects:

```bash
patator http_fuzz \
  url=http://127.0.0.1/login \
  follow=1
```

For a login experiment:

```bash
patator http_fuzz \
  url=http://127.0.0.1/login \
  method=POST \
  body='username=labuser&password=FILE0' \
  0=passwords.txt \
  follow=1
```

---

# 28. Dynamic CSRF / Nonce Testing

Some applications require a fresh token for every request.

Patator's HTTP module supports fetching a page before the main request and extracting values with regular expressions.

Example structure:

```bash
patator http_fuzz \
  url=http://127.0.0.1/login \
  method=POST \
  body='user=labuser&pass=FILE0&nonce=_N1_' \
  0=passwords.txt \
  before_urls=http://127.0.0.1/login \
  before_egrep='_N1_:name="nonce" value="(\w+)"'
```

Adapt the regular expression to your own application's HTML.

The documented module supports this pre-request/extraction workflow.

---

# 29. Basic Authentication

Patator's HTTP module supports Basic Authentication testing.

Check:

```bash
patator http_fuzz --help
```

Example:

```bash
patator http_fuzz \
  url=http://127.0.0.1/manager/ \
  user_pass=FILE0:FILE0 \
  0=users.txt \
  -x ignore:code=401
```

This tests username/password pairs in a controlled environment.

---

# 30. LDAP

Check:

```bash
patator ldap_login --help
```

Example laboratory structure:

```bash
patator ldap_login \
  host=127.0.0.1 \
  binddn='cn=FILE0,dc=example,dc=com' \
  bindpw=FILE1 \
  0=users.txt \
  1=passwords.txt
```

For LDAPS:

```bash
patator ldap_login --help
```

Configure TLS options according to your installed module.

The documented LDAP example uses `binddn`, `bindpw`, and fuzzable files.

---

# 31. SMB

Check:

```bash
patator smb_login --help
```

Laboratory example:

```bash
patator smb_login \
  host=192.168.56.101 \
  user=FILE0 \
  password=FILE1 \
  0=users.txt \
  1=passwords.txt
```

Filter normal authentication failures when appropriate:

```bash
-x ignore:fgrep=STATUS_LOGON_FAILURE
```

Full example:

```bash
patator smb_login \
  host=192.168.56.101 \
  user=FILE0 \
  password=FILE1 \
  0=users.txt \
  1=passwords.txt \
  -x ignore:fgrep=STATUS_LOGON_FAILURE
```

Only use a private Windows laboratory network.

---

# 32. MySQL

Check:

```bash
patator mysql_login --help
```

Example:

```bash
patator mysql_login \
  host=127.0.0.1 \
  user=FILE0 \
  password=FILE1 \
  0=users.txt \
  1=passwords.txt
```

For a single account:

```bash
patator mysql_login \
  host=127.0.0.1 \
  user=labuser \
  password=FILE0 \
  0=passwords.txt
```

---

# 33. PostgreSQL

Check:

```bash
patator pgsql_login --help
```

Example:

```bash
patator pgsql_login \
  host=127.0.0.1 \
  user=labuser \
  password=FILE0 \
  0=passwords.txt
```

The documented module is named `pgsql_login`.

---

# 34. MSSQL

Check:

```bash
patator mssql_login --help
```

Example:

```bash
patator mssql_login \
  host=127.0.0.1 \
  user=labuser \
  password=FILE0 \
  0=passwords.txt
```

Use a disposable database account.

---

# 35. Oracle

Check:

```bash
patator oracle_login --help
```

Example laboratory structure:

```bash
patator oracle_login \
  host=127.0.0.1 \
  user=labuser \
  password=FILE0 \
  sid=ORCL \
  0=passwords.txt
```

Oracle account lockout policies can be aggressive, so use a disposable test account and a tiny password list.

---

# 36. PostgreSQL / MySQL Comparison

PostgreSQL:

```bash
patator pgsql_login \
  host=127.0.0.1 \
  user=labuser \
  password=FILE0 \
  0=passwords.txt
```

MySQL:

```bash
patator mysql_login \
  host=127.0.0.1 \
  user=labuser \
  password=FILE0 \
  0=passwords.txt
```

The module names and parameters should always be confirmed with:

```bash
patator <module> --help
```

---

# 37. POP3

Check:

```bash
patator pop_login --help
```

Laboratory example:

```bash
patator pop_login \
  host=127.0.0.1 \
  user=FILE0 \
  password=FILE1 \
  0=users.txt \
  1=passwords.txt
```

---

# 38. IMAP

Check:

```bash
patator imap_login --help
```

Example:

```bash
patator imap_login \
  host=127.0.0.1 \
  user=FILE0 \
  password=FILE1 \
  0=users.txt \
  1=passwords.txt
```

---

# 39. VNC

Check:

```bash
patator vnc_login --help
```

Example:

```bash
patator vnc_login \
  host=127.0.0.1 \
  password=FILE0 \
  0=passwords.txt
```

Use one thread for conservative laboratory testing:

```bash
patator vnc_login \
  host=127.0.0.1 \
  password=FILE0 \
  0=passwords.txt \
  --threads 1
```

Some VNC implementations impose delays after repeated failures.

---

# 40. SNMP

Check:

```bash
patator snmp_login --help
```

SNMPv1/v2 community testing:

```bash
patator snmp_login \
  host=127.0.0.1 \
  community=FILE0 \
  0=communities.txt
```

Create:

```bash
cat > communities.txt <<'EOF'
public
private
labcommunity
EOF
```

SNMPv3 username testing:

```bash
patator snmp_login \
  host=127.0.0.1 \
  version=3 \
  user=FILE0 \
  0=users.txt
```

Use only your own SNMP laboratory service.

---

# 41. ZIP Password Recovery

Patator also contains an offline ZIP-password module.

Check:

```bash
patator unzip_pass --help
```

Example:

```bash
patator unzip_pass \
  zipfile=lab.zip \
  password=FILE0 \
  0=passwords.txt
```

This is an offline password-recovery operation against a file you are authorized to recover.

---

# 42. COMBO Files

Patator supports `COMBO` payloads.

Create:

```bash
cat > combos.txt <<'EOF'
labuser:lab123
testuser:test123
research:research123
EOF
```

Use:

```bash
patator ssh_login \
  host=127.0.0.1 \
  user=COMBO10 \
  password=COMBO11 \
  1=combos.txt
```

The `COMBO` keyword is designed for login/password entries stored together.

---

# 43. Cartesian Product

By default, Patator iterates over the Cartesian product of payload sets.

Example:

```bash
patator ssh_login \
  host=127.0.0.1 \
  user=FILE0 \
  password=FILE1 \
  0=users.txt \
  1=passwords.txt
```

If you have:

```text
4 users
7 passwords
```

the theoretical combinations are:

```text
4 × 7 = 28
```

Calculate:

```bash
echo "$(( $(wc -l < users.txt) * $(wc -l < passwords.txt) ))"
```

---

# 44. Change Testing Order

The order of `FILE` numbers controls the iteration order.

Example:

```bash
patator ssh_login \
  host=FILE2 \
  user=FILE1 \
  password=FILE0 \
  0=passwords.txt \
  1=users.txt \
  2=hosts.txt
```

This changes which payload is iterated first.

Patator documents this mechanism specifically for controlling testing order.

---

# 45. NET Keyword

Patator can generate network targets from CIDR/range notation.

Example:

```bash
patator dns_forward \
  name=lab.example \
  server=NET0 \
  0=192.168.56.0/24
```

Use network payloads only against networks you control.

The `NET` keyword supports subnet/range expansion.

---

# 46. RANGE Keyword

Patator can generate ranges.

Integer example:

```bash
patator dummy_test \
  param=RANGE0 \
  0=int:0-100
```

Hexadecimal range:

```bash
patator dummy_test \
  param=RANGE0 \
  0=hex:0x00-0xff
```

The `RANGE` keyword is useful for controlled candidate generation.

---

# 47. External Program Output

Patator can use the output of another program as a payload.

Example:

```bash
patator dummy_test \
  param=PROG0 \
  0='printf "one\ntwo\nthree\n"'
```

Check your module's behavior before using external generators.

The documented `PROG` keyword allows module parameters to consume external program output.

---

# 48. Ignore Conditions

Ignore a specific response:

```bash
-x ignore:code=404
```

Ignore a message:

```bash
-x ignore:fgrep='Invalid credentials'
```

Ignore a regular expression match:

```bash
-x ignore:egrep='Invalid|Failed'
```

Conditions within the same `-x` expression are combined according to Patator's condition syntax. Multiple `-x` options can be used for separate actions.

---

# 49. Retry Conditions

Retry on HTTP 500:

```bash
-x ignore,retry:code=500
```

Retry behavior can also be used when a service temporarily disconnects or rate-limits the client.

Check:

```bash
patator --help
```

before changing retry limits.

---

# 50. Skip After Success

Skip further payloads for a particular value:

```bash
-x skip=0:fgrep=Success
```

Example:

```bash
patator dummy_test \
  data=FILE0.FILE1 \
  0=users.txt \
  1=passwords.txt \
  -x skip=0:fgrep=Success
```

This can reduce unnecessary testing after a valid result is found.

---

# 51. Free After Success

Stop testing a combination after success:

```bash
-x free=data:fgrep=Success
```

For SSH host testing:

```bash
-x free=host:code=0
```

For host + user:

```bash
-x free=host+user:code=0
```

Use these controls to avoid unnecessary authentication attempts.

---

# 52. Threads

Check global options:

```bash
patator --help
```

Run a conservative laboratory test:

```bash
patator ssh_login \
  host=127.0.0.1 \
  user=labuser \
  password=FILE0 \
  0=passwords.txt \
  --threads 1
```

Increase carefully:

```bash
--threads 2
```

Then:

```bash
--threads 4
```

High concurrency can cause connection failures or trigger service protections.

---

# 53. Rate Limiting

Patator provides rate-control options in current builds.

Check:

```bash
patator --help
```

For a laboratory experiment, deliberately use a low request rate and measure:

```text
Attempts
Time
Response latency
Rate-limit response
Lockout
Server load
```

Never use rate-control settings to bypass security controls on systems you do not own.

---

# 54. Server Logs

While testing SSH:

```bash
sudo journalctl -u ssh -f
```

On systems using `auth.log`:

```bash
sudo tail -f /var/log/auth.log
```

For a web application:

```bash
sudo tail -f /var/log/nginx/access.log
```

or:

```bash
sudo tail -f /var/log/apache2/access.log
```

Compare:

```text
Patator attempt
      ↓
Authentication request
      ↓
Server log
      ↓
Detection rule
      ↓
Alert
```

---

# 55. Lockout Testing

Create a deliberately small list:

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
patator ssh_login \
  host=127.0.0.1 \
  user=labuser \
  password=FILE0 \
  0=bad-passwords.txt \
  --threads 1
```

Record:

```text
Failed attempts
Lockout threshold
Lockout duration
Server response
Log entry
Alert
```

Use a disposable account.

---

# 56. Rate-Limit Testing

Start with:

```bash
--threads 1
```

Record:

```text
Requests/minute
Average response time
Failed attempts
Rate-limit threshold
```

Then compare with:

```bash
--threads 2
```

and:

```bash
--threads 4
```

Keep the experiment small and isolated.

---

# 57. Authentication Detection

A useful defensive test is:

```text
Patator
   ↓
Repeated authentication failures
   ↓
Application / service logs
   ↓
Detection engine
   ↓
Alert
```

Check whether your monitoring detects:

```text
Repeated failures
Rapid authentication
Multiple usernames
Multiple passwords
Successful login after failures
Account lockout
Repeated source IP
```

---

# 58. HTTP Failure Analysis

Before automating a web login, test it manually:

```bash
curl -i http://127.0.0.1/login
```

Submit an invalid credential:

```bash
curl -i \
  -X POST \
  -d 'username=labuser&password=wrong' \
  http://127.0.0.1/login
```

Record:

```text
HTTP status
Response body
Response size
Redirect
Cookie
Failure message
```

Then test the known valid laboratory credential.

Use the difference to configure Patator's response conditions.

---

# 59. Output Logging

Create:

```bash
mkdir -p experiments
```

Run:

```bash
patator ssh_login \
  host=127.0.0.1 \
  user=labuser \
  password=FILE0 \
  0=passwords.txt \
  --threads 1 \
  2>&1 | tee experiments/ssh-test-01.log
```

Read:

```bash
less experiments/ssh-test-01.log
```

---

# 60. Experiment Matrix

Record each test:

```text
Service | Threads | Users | Passwords | Result | Time
-------------------------------------------------------
SSH     | 1       | 1     | 10        | ...    | ...
SSH     | 2       | 1     | 10        | ...    | ...
FTP     | 1       | 1     | 10        | ...    | ...
HTTP    | 1       | 1     | 10        | ...    | ...
```

Keep these constant when comparing performance:

```text
Target
Credential dataset
Network
Service configuration
```

---

# 61. Troubleshooting

### Command Not Found

```bash
which patator
```

Install:

```bash
sudo apt update
sudo apt install patator
```

---

### Module Not Found

Check:

```bash
patator --help
```

Then:

```bash
patator <module> --help
```

Module availability depends on the installed Patator version.

---

### Authentication Always Fails

Test manually first.

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

If manual authentication fails, fix the laboratory service first.

---

### Too Many Connections

Reduce:

```bash
--threads 1
```

Then check the service logs.

For SSH:

```bash
sudo journalctl -u ssh -f
```

Patator's documentation specifically notes that SSH/FTP/Telnet services can enforce concurrent-connection limits.

---

### False Results

Check the response condition.

For HTTP, compare:

```text
Valid response
Invalid response
```

For FTP/SSH, inspect the actual failure message and configure the appropriate `-x ignore:` condition.

---

# 62. Practical Progression

```text
Level 1
Install
Help
Modules
FILE syntax
Single username
Small password list

        ↓

Level 2
Username lists
Password lists
SSH
FTP
Telnet
SMTP

        ↓

Level 3
HTTP
POST forms
Cookies
Redirects
LDAP
SMB
Databases

        ↓

Level 4
COMBO
NET
RANGE
PROG
Cartesian products
Iteration order

        ↓

Level 5
Ignore conditions
Retry
Skip
Free
Threads
Rate limiting

        ↓

Level 6
Dynamic HTTP tokens
Authentication detection
Lockout research
Service comparison
Repeatable experiments
```

---

# 63. Quick Reference

```bash
# General help
patator --help

# Module help
patator ssh_login --help

# SSH single user
patator ssh_login host=127.0.0.1 user=labuser password=FILE0 0=passwords.txt

# SSH username + password lists
patator ssh_login host=127.0.0.1 user=FILE0 password=FILE1 0=users.txt 1=passwords.txt

# SSH single mode
patator ssh_login host=127.0.0.1 user=FILE0 password=FILE0 0=users.txt

# FTP
patator ftp_login host=127.0.0.1 user=FILE0 password=FILE1 0=users.txt 1=passwords.txt

# SMTP
patator smtp_login host=127.0.0.1 user=FILE0 password=FILE1 0=users.txt 1=passwords.txt

# HTTP
patator http_fuzz url=http://127.0.0.1/FILE0 0=words.txt

# HTTP POST
patator http_fuzz \
  url=http://127.0.0.1/login \
  method=POST \
  body='username=labuser&password=FILE0' \
  0=passwords.txt

# SMB
patator smb_login host=192.168.56.101 user=FILE0 password=FILE1 0=users.txt 1=passwords.txt

# MySQL
patator mysql_login host=127.0.0.1 user=labuser password=FILE0 0=passwords.txt

# PostgreSQL
patator pgsql_login host=127.0.0.1 user=labuser password=FILE0 0=passwords.txt

# LDAP
patator ldap_login host=127.0.0.1 binddn='cn=FILE0,dc=example,dc=com' bindpw=FILE1 0=users.txt 1=passwords.txt

# ZIP password recovery
patator unzip_pass zipfile=lab.zip password=FILE0 0=passwords.txt

# Threads
--threads 1

# Stop testing a host after success
-x free=host:code=0

# Stop testing a host/user after success
-x free=host+user:code=0
```

---

# 64. Final Laboratory Workflow

```text
Create Isolated Service
        ↓
Create Disposable Account
        ↓
Create Small Wordlists
        ↓
Verify Authentication Manually
        ↓
Check Patator Module
        ↓
Build FILE Payloads
        ↓
Run Small Test
        ↓
Verify Failure Conditions
        ↓
Monitor Server Logs
        ↓
Test Lockout / Rate Limiting
        ↓
Increase Dataset Carefully
        ↓
Record Results
        ↓
Document Findings
```

---

# Scope

Patator performs authentication testing and fuzzing against network services.

Use it only against:

* Your own systems
* Isolated laboratory services
* CTF environments
* Explicitly authorized penetration-testing targets

Do not use these commands against third-party accounts, public login portals, or systems without authorization.
