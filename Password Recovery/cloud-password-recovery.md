# Cloud Password Recovery

> Practical laboratory guide for researching password recovery and credential security in authorized cloud environments.

---

## Scope

This guide covers:

```text
Cloud IAM credentials
Password hashes
Offline password recovery
Cloud identity stores
Credential inventories
Password-policy testing
John the Ripper
Hashcat
Wordlists
Rules
Masks
Hybrid attacks
Credential exposure research
Recovery verification
```

The main principle is:

```text
Cloud Account
     ↓
Authorized Credential Material
     ↓
Offline Analysis
     ↓
Hash Identification
     ↓
Recovery Testing
     ↓
Verification
     ↓
Report
```

Do not perform password guessing, credential spraying, or brute-force authentication against live cloud services unless you have explicit authorization and defined testing limits.

---

# 1. Lab Requirements

Recommended:

```text
Linux VM
Python 3
John the Ripper
Hashcat
SQLite
Cloud test account
Synthetic credentials
```

Install:

```bash
sudo apt update
sudo apt install john hashcat sqlite3 python3
```

Check:

```bash
john --version
hashcat --version
python3 --version
```

---

# 2. Cloud Password Recovery vs Cloud Account Attacks

These are different activities.

### Offline recovery

```text
Obtained authorized hash
        ↓
John / Hashcat
        ↓
Candidate testing
```

### Online authentication attack

```text
Username
   ↓
Cloud login endpoint
   ↓
Password guesses
```

This guide focuses primarily on the first workflow.

For authorized online assessments, follow the cloud provider's penetration-testing rules and the engagement's rate limits.

---

# 3. Create a Cloud Laboratory

Use a dedicated test environment.

Recommended:

```text
Organization:
security-lab

User:
password-research

Password:
Known laboratory password
```

Use:

```text
Disposable account
Dedicated test tenant/project
Dedicated test credentials
```

Do not use:

```text
Production account
Personal account
Banking account
Company administrator account
Real customer credentials
```

---

# 4. Record the Environment

Document:

```text
Cloud provider
Tenant / organization
Test account
Account role
Password policy
MFA configuration
Region
Testing window
Authorization
```

Example:

```text
Provider: <cloud-provider>
Environment: Laboratory
Account: password-research
Role: Test User
MFA: Enabled
```

---

# 5. Password Policy

Before testing password recovery, inspect the organization's password policy.

Record:

```text
Minimum length
Maximum length
Character requirements
Password history
Expiration
Lockout
MFA
Compromised-password controls
```

The exact policy depends on the cloud identity platform.

---

# 6. Synthetic Password Dataset

Create a local dataset for password-security research.

Example:

```bash
cat > cloud-passwords.txt <<'EOF'
cloud
cloud123
cloudlab
cloudlab123
CloudLab123
CloudLab2026
Cloud-Lab-2026!
research
research123
EOF
```

Check:

```bash
wc -l cloud-passwords.txt
```

---

# 7. Generate Laboratory Hashes

For a simple laboratory SHA-256 test:

```bash
printf 'CloudLab123' | sha256sum
```

Save:

```bash
printf 'CloudLab123' |
    sha256sum |
    awk '{print $1}' > cloud.hash
```

Check:

```bash
cat cloud.hash
```

This is a synthetic hash and is suitable for testing John/Hashcat workflows.

---

# 8. John the Ripper

Check formats:

```bash
john --list=formats
```

Search:

```bash
john --list=formats | grep -Ei 'sha|bcrypt|argon|pbkdf'
```

---

# 9. John Dictionary Attack

For the synthetic SHA-256 example:

```bash
john \
    --format=raw-sha256 \
    --wordlist=cloud-passwords.txt \
    cloud.hash
```

Show:

```bash
john \
    --format=raw-sha256 \
    --show \
    cloud.hash
```

---

# 10. John Rules

Run:

```bash
john \
    --format=raw-sha256 \
    --wordlist=cloud-passwords.txt \
    --rules \
    cloud.hash
```

Show:

```bash
john --show cloud.hash
```

Rules can generate password variations from a smaller base wordlist.

---

# 11. John Incremental

Check:

```bash
john --list=incremental
```

Run only against your laboratory hash:

```bash
john \
    --format=raw-sha256 \
    --incremental \
    cloud.hash
```

Status:

```bash
john --status
```

Stop:

```text
Ctrl+C
```

---

# 12. Hashcat

Check the correct mode:

```bash
hashcat --example-hashes | grep -i 'sha-256'
```

For raw SHA-256:

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    cloud-passwords.txt
```

Show:

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    --show
```

Always verify the mode against your installed Hashcat version.

---

# 13. Hashcat Rules

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    cloud-passwords.txt \
    -r /usr/share/hashcat/rules/best64.rule
```

Show:

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    --show
```

---

# 14. Hashcat Mask Attack

Test a known laboratory pattern:

```text
Cloud + four digits
```

Run:

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    -a 3 \
    'Cloud?d?d?d?d'
```

This is useful for controlled password-pattern research.

---

# 15. Custom Character Set

Example:

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    -a 3 \
    -1 '?l?d' \
    '?1?1?1?1?1?1'
```

This tests a six-character laboratory keyspace consisting of lowercase letters and digits.

---

# 16. Hybrid Attack

Word + two digits:

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    -a 6 \
    cloud-passwords.txt \
    '?d?d'
```

Two digits + word:

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    -a 7 \
    '?d?d' \
    cloud-passwords.txt
```

---

# 17. Candidate-Space Research

Four digits:

```bash
seq -w 0000 9999 > four-digit.txt
```

Count:

```bash
wc -l four-digit.txt
```

Expected:

```text
10000
```

Run:

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    four-digit.txt
```

This gives a controlled experiment for a 10,000-candidate keyspace.

---

# 18. Password-Length Experiment

Create synthetic passwords:

```text
cloud1
cloud123
Cloud123
CloudPassword1
CloudPassword2026
Cloud-Password-2026!
```

Generate hashes.

Record:

```text
Password
Length
Character classes
Candidate space
Recovery time
```

Compare the results.

---

# 19. Password-Policy Experiment

Create datasets representing different policies:

```text
8-character
10-character
12-character
16-character
20-character
```

Then compare:

```text
Lowercase only
Lowercase + digits
Mixed case + digits
Mixed case + symbols
```

Record:

```text
Length
Character set
Total keyspace
Attack type
Speed
Estimated search time
```

---

# 20. Cloud Credential Inventory

In an authorized assessment, create an inventory without collecting unnecessary secrets.

Record:

```text
Identity
Role
Authentication method
MFA
Password policy
Credential type
Last rotation
Credential status
```

Example:

```text
User: password-research
Role: Test User
MFA: Enabled
Password Auth: Enabled
```

Do not store real passwords in the research repository.

---

# 21. Password Hashes vs Cloud Credentials

Cloud environments can contain different credential types:

```text
Password
Password hash
API key
Access token
Refresh token
SSH key
Service-account credential
Application secret
```

These are not interchangeable.

A password hash can be tested offline.

An access token is not normally a password hash.

An API key is not automatically suitable for John or Hashcat.

---

# 22. API Keys

Treat API keys as secrets.

Do not attempt to "crack" an API key as though it were a password hash.

For authorized research, instead document:

```text
Key type
Key length
Storage location
Permissions
Expiration
Rotation
Exposure
Revocation status
```

If a laboratory API key is intentionally exposed, rotate or revoke it after the experiment.

---

# 23. Access Tokens

Access tokens can represent active authentication.

Never publish:

```text
Access tokens
Refresh tokens
Session tokens
Bearer tokens
```

For laboratory research, use deliberately generated test credentials.

Document:

```text
Token type
Lifetime
Scope
Audience
Revocation behavior
```

---

# 24. Password Hash Extraction

If an authorized cloud identity system provides password hashes for a security assessment, preserve the original data.

Create:

```bash
mkdir -p cloud-lab/{evidence,hashes,wordlists,results}
```

Store the original separately:

```text
cloud-lab/
├── evidence/
├── hashes/
├── wordlists/
└── results/
```

Never modify the original evidence.

---

# 25. Evidence Hashing

Calculate:

```bash
sha256sum authorized-hashes.txt
```

Save:

```bash
sha256sum authorized-hashes.txt \
    > cloud-lab/evidence/SHA256SUMS.txt
```

Verify later:

```bash
sha256sum -c cloud-lab/evidence/SHA256SUMS.txt
```

---

# 26. Hash Identification

Inspect:

```bash
head authorized-hashes.txt
```

Use:

```bash
hashid '<HASH>'
```

If installed:

```bash
hash-identifier
```

Hashcat:

```bash
hashcat --example-hashes
```

John:

```bash
john --list=formats
```

Confirm the algorithm from authoritative application or cloud documentation whenever possible.

---

# 27. Salted Cloud Hashes

If a cloud application provides salted hashes, preserve:

```text
Algorithm
Salt
Derived value
Cost
Iterations
Memory
Parallelism
```

Do not remove fields from the original representation.

Example conceptual structure:

```text
algorithm$salt$derived-value
```

The exact format depends on the identity system.

---

# 28. Slow Password Hashing

Modern password-storage systems may use intentionally expensive functions such as:

```text
bcrypt
scrypt
Argon2
PBKDF2
```

These are designed differently from:

```text
MD5
SHA-1
SHA-256
SHA-512
```

Do not compare them simply by hash length.

For research, record:

```text
Algorithm
Cost
Iterations
Memory
Parallelism
Salt
```

---

# 29. Benchmarking

John:

```bash
john --test
```

Hashcat:

```bash
hashcat -b
```

For a specific Hashcat mode:

```bash
hashcat \
    -m <MODE> \
    -b
```

Record:

```text
Tool
Version
CPU
GPU
Driver
Algorithm
Mode
Speed
```

---

# 30. Compare Recovery Methods

Run the same synthetic hash using:

```text
Dictionary
Dictionary + Rules
Mask
Hybrid
Incremental
```

Use the same:

```text
Hash
Password
Hardware
Candidate space
```

Record:

```text
Attack
Candidates
Speed
Runtime
Result
```

This makes the experiment reproducible.

---

# 31. Cloud Password Spray Research

Password spraying is an online authentication technique and should not be performed against arbitrary cloud tenants.

For an authorized engagement:

```text
Defined test account
Defined password
Approved IP address
Approved testing window
Provider-approved methodology
Rate limit
Stop condition
```

Document authorization before testing.

Do not include real password-spraying commands in a public research repository.

---

# 32. MFA Research

Password recovery does not automatically defeat MFA.

Record:

```text
Password recovered
MFA enabled
MFA method
Authentication result
```

A successful offline password recovery should not be interpreted as successful account compromise when MFA remains enforced.

---

# 33. Password Reuse Research

In a controlled lab, create separate test services:

```text
Cloud
VPN
Git
Email
Internal application
```

Use deliberately weak test passwords.

Document whether the same password is reused.

The goal is to demonstrate the risk of password reuse without testing real accounts.

---

# 34. Credential Exposure Research

Create a synthetic repository:

```text
cloud-lab/
└── synthetic-exposure/
    ├── users.txt
    ├── hashes.txt
    └── report.md
```

Test:

```text
Password exposure
Password reuse
Weak passwords
Predictable patterns
Credential rotation
MFA coverage
```

Do not commit actual cloud credentials.

---

# 35. Credential Rotation

After a laboratory experiment:

```text
1. Rotate test password.
2. Revoke test API keys.
3. Revoke test sessions.
4. Delete temporary credentials.
5. Remove test users if no longer required.
```

Verify that old credentials no longer work.

---

# 36. Recovery Verification

A recovered password should be verified only against the authorized laboratory account.

Document:

```text
Hash recovered: Yes
Password verified: Yes
MFA required: Yes/No
Account: Laboratory account
```

Never test recovered credentials against unrelated services.

---

# 37. Logging John

```bash
john \
    --format=raw-sha256 \
    --wordlist=cloud-passwords.txt \
    cloud.hash \
    2>&1 | tee cloud-lab/results/john.log
```

Show:

```bash
john --show cloud.hash
```

---

# 38. Logging Hashcat

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    cloud-passwords.txt \
    2>&1 | tee cloud-lab/results/hashcat.log
```

Show:

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    --show
```

---

# 39. Hashcat Sessions

```bash
hashcat \
    --session=cloud-lab \
    -m 1400 \
    cloud.hash \
    cloud-passwords.txt
```

Status:

```bash
hashcat \
    --session=cloud-lab \
    --status
```

Resume:

```bash
hashcat \
    --session=cloud-lab \
    --restore
```

---

# 40. Results Template

Record each experiment:

```text
Experiment:
Date:
Cloud Platform:
Environment:
Hash Algorithm:
Hash Format:
Salt:
Cost:
Attack:
Wordlist:
Candidate Count:
Tool:
Version:
CPU:
GPU:
Speed:
Runtime:
Recovered:
Verified:
```

---

# 41. Example Research Table

```text
Algorithm | Attack | Candidates | Tool | Result
----------|--------|------------|------|-------
SHA-256   | Dict   | 10,000     | John | Recovered
SHA-256   | Rules  | 10,000+    | John | Recovered
SHA-256   | Mask   | 10,000     | Hashcat | Recovered
bcrypt    | Dict   | 10,000     | Hashcat | Measured
```

Replace the example results with measurements from your own laboratory.

---

# 42. Troubleshooting

## Hashcat Does Not Recognize the Hash

Check:

```bash
hashcat --example-hashes
```

Verify:

```text
Algorithm
Encoding
Salt
Cost
Hash format
Mode
```

---

## John Does Not Recognize the Hash

Run:

```bash
john --list=formats
```

Search:

```bash
john --list=formats | grep -Ei 'sha|bcrypt|argon|pbkdf'
```

---

## Password Not Found

Possible causes:

```text
Wrong algorithm
Wrong format
Wrong extraction
Wrong wordlist
Wrong mask
Password outside candidate space
Incorrect salt
Incorrect cost parameters
```

Validate the workflow with a known laboratory password.

---

# 43. Git Security

Never commit:

```text
Passwords
Password hashes from real users
API keys
Access tokens
Refresh tokens
Cloud credentials
SSH private keys
Browser cookies
Production database dumps
```

Recommended `.gitignore`:

```gitignore
*.hash
*.pot
*.potfile
*.sql
*.dump
*.db
*.pem
*.key
*.token

cloud-lab/evidence/
cloud-lab/results/
```

Synthetic test data may be committed when it contains no real secrets.

---

# 44. Complete Practical Workflow

```text
Authorized Cloud Lab
        ↓
Create Test Identity
        ↓
Define Password Policy
        ↓
Generate Synthetic Hashes
        ↓
Identify Algorithm
        ↓
Preserve Evidence
        ↓
Select John / Hashcat
        ↓
Dictionary
        ↓
Rules
        ↓
Mask
        ↓
Hybrid
        ↓
Benchmark
        ↓
Verify
        ↓
Rotate Credentials
        ↓
Document Results
```

---

# 45. Quick Reference

### Install

```bash
sudo apt install john hashcat sqlite3 python3
```

### John Formats

```bash
john --list=formats
```

### Hashcat Modes

```bash
hashcat --example-hashes
```

### SHA-256 Test Hash

```bash
printf 'CloudLab123' | sha256sum
```

### John Dictionary

```bash
john \
    --format=raw-sha256 \
    --wordlist=cloud-passwords.txt \
    cloud.hash
```

### John Rules

```bash
john \
    --format=raw-sha256 \
    --wordlist=cloud-passwords.txt \
    --rules \
    cloud.hash
```

### John Result

```bash
john --show cloud.hash
```

### Hashcat Dictionary

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    cloud-passwords.txt
```

### Hashcat Rules

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    cloud-passwords.txt \
    -r /usr/share/hashcat/rules/best64.rule
```

### Hashcat Mask

```bash
hashcat \
    -m 1400 \
    cloud.hash \
    -a 3 \
    'Cloud?d?d?d?d'
```

### Benchmark

```bash
john --test
hashcat -b
```

---

# 46. Research Checklist

```text
[ ] Dedicated cloud laboratory created
[ ] Test identity created
[ ] Test password created
[ ] MFA configuration documented
[ ] Password policy documented
[ ] Authorization confirmed
[ ] Synthetic hashes generated
[ ] Hash algorithm identified
[ ] Hash format verified
[ ] Salt documented
[ ] Cost parameters documented
[ ] John format verified
[ ] Hashcat mode verified
[ ] Dictionary attack tested
[ ] Rules tested
[ ] Mask tested
[ ] Hybrid tested
[ ] Benchmark completed
[ ] Recovery verified
[ ] Test credentials rotated
[ ] API keys revoked
[ ] Sessions revoked
[ ] Real credentials excluded
[ ] Secrets excluded from Git
[ ] Results documented
```

---

## Scope

Use this guide for:

* Your own cloud accounts
* Dedicated cloud-security laboratories
* Authorized penetration tests
* CTF environments
* Synthetic password datasets
* Offline password-hash research
* Cloud IAM security research

Do not use recovered credentials, API keys, access tokens, or session material against systems or accounts without explicit authorization.
