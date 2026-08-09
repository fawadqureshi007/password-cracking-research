# Database Password Recovery

> Practical laboratory guide for identifying, extracting, and testing password hashes obtained from authorized database security assessments.

---

## Scope

This guide covers:

```text
Database password hashes
Hash identification
Hash extraction
John the Ripper
Hashcat
Wordlist attacks
Rule-based attacks
Mask attacks
Hybrid attacks
Salted hashes
Parameterized hashes
Benchmarking
Recovery verification
```

The workflow is:

```text
Database / Dump
       ↓
Identify Hash Location
       ↓
Extract Hashes
       ↓
Identify Hash Type
       ↓
Select John / Hashcat Mode
       ↓
Run Offline Recovery
       ↓
Verify Results
       ↓
Document Findings
```

Only use credentials and password hashes from systems you own or have explicit authorization to assess.

---

# 1. Lab Requirements

Recommended:

```text
Kali Linux
John the Ripper
Hashcat
Python 3
SQLite
PostgreSQL client
MySQL/MariaDB client
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
sqlite3 --version
python3 --version
```

---

# 2. Create a Laboratory Database

Create a working directory:

```bash
mkdir database-lab
cd database-lab
```

Create a SQLite database:

```bash
sqlite3 lab.db
```

Create a table:

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    username TEXT,
    password_hash TEXT
);
```

Exit:

```text
.quit
```

---

# 3. Populate Test Data

For research purposes, use deliberately generated test hashes.

Example:

```bash
sqlite3 lab.db <<'EOF'
INSERT INTO users (username, password_hash)
VALUES
('alice', 'LAB_HASH_01'),
('bob', 'LAB_HASH_02'),
('charlie', 'LAB_HASH_03');
EOF
```

Check:

```bash
sqlite3 lab.db "SELECT * FROM users;"
```

---

# 4. Hash Types

Common password-storage formats you may encounter include:

```text
MD5
SHA-1
SHA-256
SHA-512
bcrypt
scrypt
Argon2
PBKDF2
NTLM
```

These are not equally resistant to offline password recovery.

Do not identify a hash solely from its length.

---

# 5. Identify Hashes

John:

```bash
john --list=formats
```

Hashcat:

```bash
hashcat --help
```

Hashcat example hashes:

```bash
hashcat --example-hashes
```

Search:

```bash
hashcat --example-hashes | grep -Ei 'md5|sha1|sha256|sha512|bcrypt|scrypt|argon'
```

For ambiguous formats, examine:

```text
Prefix
Length
Separators
Salt
Cost parameters
Encoding
Database/application format
```

---

# 6. Hash Identification Tools

If available:

```bash
hashid '<HASH>'
```

or:

```bash
hash-identifier
```

Do not blindly trust automatic identification.

Confirm the format from the application or database schema whenever possible.

---

# 7. Extract Hashes from SQLite

Inspect the database:

```bash
sqlite3 lab.db ".tables"
```

View schema:

```bash
sqlite3 lab.db ".schema users"
```

Extract:

```bash
sqlite3 -noheader lab.db \
    "SELECT username || ':' || password_hash FROM users;" \
    > hashes.txt
```

Check:

```bash
cat hashes.txt
```

---

# 8. Separate Username and Hash

Example input:

```text
alice:HASH
bob:HASH
charlie:HASH
```

If your cracking tool expects only hashes:

```bash
cut -d: -f2 hashes.txt > password-hashes.txt
```

Check:

```bash
cat password-hashes.txt
```

Always preserve the original evidence separately.

---

# 9. Preserve the Original Dump

Create:

```bash
mkdir evidence
cp lab.db evidence/
cp hashes.txt evidence/
```

Calculate hashes:

```bash
sha256sum evidence/lab.db
sha256sum evidence/hashes.txt
```

Save the results:

```bash
sha256sum evidence/* > evidence/SHA256SUMS.txt
```

This helps detect accidental modification during research.

---

# 10. Plain MD5 Laboratory Test

Generate a known test hash:

```bash
printf 'labpassword' | md5sum
```

Save it:

```bash
printf 'labpassword' | md5sum | awk '{print $1}' > md5.hash
```

Create a wordlist:

```bash
cat > passwords.txt <<'EOF'
password
password123
lab
lab123
labpassword
research
research123
EOF
```

---

# 11. John MD5 Test

Run:

```bash
john \
    --format=raw-md5 \
    --wordlist=passwords.txt \
    md5.hash
```

Show:

```bash
john \
    --format=raw-md5 \
    --show \
    md5.hash
```

---

# 12. Hashcat MD5 Test

Use the corresponding mode from your installed version.

Typical raw MD5 mode:

```bash
hashcat \
    -m 0 \
    md5.hash \
    passwords.txt
```

Show:

```bash
hashcat \
    -m 0 \
    md5.hash \
    --show
```

Verify the mode:

```bash
hashcat --example-hashes | grep -i 'MD5'
```

---

# 13. SHA-1 Laboratory Test

Generate:

```bash
printf 'labpassword' | sha1sum
```

Save:

```bash
printf 'labpassword' | sha1sum | awk '{print $1}' > sha1.hash
```

John:

```bash
john \
    --format=raw-sha1 \
    --wordlist=passwords.txt \
    sha1.hash
```

Hashcat:

```bash
hashcat \
    -m 100 \
    sha1.hash \
    passwords.txt
```

Show:

```bash
hashcat \
    -m 100 \
    sha1.hash \
    --show
```

---

# 14. SHA-256 Laboratory Test

Generate:

```bash
printf 'labpassword' | sha256sum
```

Save:

```bash
printf 'labpassword' | sha256sum | awk '{print $1}' > sha256.hash
```

John:

```bash
john \
    --format=raw-sha256 \
    --wordlist=passwords.txt \
    sha256.hash
```

Hashcat:

```bash
hashcat \
    -m 1400 \
    sha256.hash \
    passwords.txt
```

Show:

```bash
hashcat \
    -m 1400 \
    sha256.hash \
    --show
```

---

# 15. SHA-512 Laboratory Test

Generate:

```bash
printf 'labpassword' | sha512sum
```

Save:

```bash
printf 'labpassword' | sha512sum | awk '{print $1}' > sha512.hash
```

John:

```bash
john \
    --format=raw-sha512 \
    --wordlist=passwords.txt \
    sha512.hash
```

Hashcat:

```bash
hashcat \
    -m 1700 \
    sha512.hash \
    passwords.txt
```

---

# 16. Salted Hashes

A salted password hash generally contains information similar to:

```text
salt + password
```

The exact representation depends on the application.

Example conceptual format:

```text
algorithm$salt$derived-value
```

Do not remove the salt before attempting recovery.

The salt is part of the verification scheme.

---

# 17. Identify Salted Formats

Inspect:

```bash
cat hashes.txt
```

Look for:

```text
$
:
*
+
```

or other separators.

Examples of recognizable structures may include:

```text
$2...
$argon2...
$pbkdf2...
```

The exact format determines the required cracking mode.

---

# 18. bcrypt Laboratory Test

Create a bcrypt hash using Python.

Install the library if necessary:

```bash
python3 -m pip install bcrypt
```

Generate:

```bash
python3 - <<'PY'
import bcrypt

password = b"labpassword"
hashed = bcrypt.hashpw(password, bcrypt.gensalt())

print(hashed.decode())
PY
```

Copy the resulting hash into:

```bash
nano bcrypt.hash
```

---

# 19. John bcrypt Test

Use the bcrypt format supported by your John build:

```bash
john \
    --wordlist=passwords.txt \
    bcrypt.hash
```

Show:

```bash
john --show bcrypt.hash
```

If John does not automatically detect it:

```bash
john --list=formats | grep -i bcrypt
```

---

# 20. Hashcat bcrypt Test

Find the appropriate mode:

```bash
hashcat --example-hashes | grep -i bcrypt
```

Then:

```bash
hashcat \
    -m <BCRYPT_MODE> \
    bcrypt.hash \
    passwords.txt
```

Show:

```bash
hashcat \
    -m <BCRYPT_MODE> \
    bcrypt.hash \
    --show
```

Do not assume the mode from an old guide; verify it against your installed Hashcat version.

---

# 21. Password Hashes in SQL Dumps

Suppose an authorized SQL dump contains:

```sql
INSERT INTO users VALUES
(1,'alice','HASH1'),
(2,'bob','HASH2');
```

Locate likely password fields:

```bash
grep -Ein 'password|passwd|pass_hash|password_hash' database.sql
```

Extract the relevant records manually or with a parser.

Do not upload an actual production dump to a public repository.

---

# 22. Search a Dump for Hashes

Basic search:

```bash
grep -Ein 'password_hash|passwd|password' database.sql
```

Search common hash lengths:

```bash
grep -Eio '[a-f0-9]{32}' database.sql
```

SHA-1-like values:

```bash
grep -Eio '[a-f0-9]{40}' database.sql
```

SHA-256-like values:

```bash
grep -Eio '[a-f0-9]{64}' database.sql
```

These searches are only candidate extraction methods.

Hash length alone does not prove the algorithm.

---

# 23. Extract Candidate Hashes

For a controlled MD5 dump:

```bash
grep -Eio '[a-f0-9]{32}' database.sql \
    | sort -u \
    > extracted-md5.txt
```

Check:

```bash
wc -l extracted-md5.txt
head extracted-md5.txt
```

Verify candidates before cracking.

---

# 24. Wordlist Attack

Basic John workflow:

```bash
john \
    --wordlist=passwords.txt \
    password-hashes.txt
```

Show:

```bash
john --show password-hashes.txt
```

Hashcat:

```bash
hashcat \
    -m <MODE> \
    password-hashes.txt \
    passwords.txt
```

---

# 25. Rules

John:

```bash
john \
    --wordlist=passwords.txt \
    --rules \
    password-hashes.txt
```

Hashcat:

```bash
hashcat \
    -m <MODE> \
    password-hashes.txt \
    passwords.txt \
    -r /usr/share/hashcat/rules/best64.rule
```

Rules are particularly useful when passwords are variations of dictionary words.

---

# 26. Mask Attack

For a controlled four-digit password:

```bash
hashcat \
    -m <MODE> \
    password-hashes.txt \
    -a 3 \
    '?d?d?d?d'
```

Example pattern:

```text
LAB + four digits
```

```bash
hashcat \
    -m <MODE> \
    password-hashes.txt \
    -a 3 \
    'LAB?d?d?d?d'
```

Only use masks appropriate to the laboratory password policy or known test pattern.

---

# 27. Custom Character Sets

Example six-character lowercase/digit search:

```bash
hashcat \
    -m <MODE> \
    password-hashes.txt \
    -a 3 \
    -1 '?l?d' \
    '?1?1?1?1?1?1'
```

The candidate space increases rapidly as password length and character diversity increase.

---

# 28. Hybrid Attack

Word + two digits:

```bash
hashcat \
    -m <MODE> \
    password-hashes.txt \
    -a 6 \
    passwords.txt \
    '?d?d'
```

Two digits + word:

```bash
hashcat \
    -m <MODE> \
    password-hashes.txt \
    -a 7 \
    '?d?d' \
    passwords.txt
```

---

# 29. John Incremental Mode

Check:

```bash
john --list=incremental
```

Run:

```bash
john \
    --incremental \
    password-hashes.txt
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

# 30. Candidate-Space Experiment

Four digits:

```bash
seq -w 0000 9999 > 4digit.txt
```

Count:

```bash
wc -l 4digit.txt
```

Total:

```text
10,000 candidates
```

Run:

```bash
john \
    --wordlist=4digit.txt \
    password-hashes.txt
```

This is useful for controlled benchmarking.

---

# 31. Password-Length Research

Create laboratory passwords:

```text
lab1
lab123
lab1234
LabPass123
LabPassword2026
Lab-Password-2026!
```

Generate their hashes using the appropriate algorithm.

Record:

```text
Length
Character classes
Algorithm
Salt
Cost
Candidate space
Recovery time
```

---

# 32. Hash-Speed Comparison

Create equivalent laboratory passwords using:

```text
MD5
SHA-1
SHA-256
SHA-512
bcrypt
```

Benchmark them separately.

Example:

```bash
hashcat -m 0 -b
```

Then benchmark the relevant modes:

```bash
hashcat -m <MODE> -b
```

Record:

```text
Algorithm
Mode
Hardware
Hash rate
```

Do not directly compare algorithms without considering their intended password-storage design and cost parameters.

---

# 33. John Benchmark

Run:

```bash
john --test
```

List formats:

```bash
john --list=formats
```

For a particular format:

```bash
john \
    --test \
    --format=<FORMAT>
```

Record the results.

---

# 34. Hashcat Benchmark

Run:

```bash
hashcat -b
```

For one mode:

```bash
hashcat \
    -m <MODE> \
    -b
```

Record:

```text
Hashcat version
Driver
GPU
CPU
Mode
Speed
```

---

# 35. Database Extraction Workflow

For an authorized database assessment:

```text
Database / SQL Dump
        ↓
Identify User Table
        ↓
Identify Password Column
        ↓
Determine Hash Representation
        ↓
Extract Hashes
        ↓
Preserve Original Evidence
        ↓
Identify Algorithm
        ↓
Select John / Hashcat
        ↓
Test Wordlist
        ↓
Rules
        ↓
Mask / Hybrid
        ↓
Verify Results
        ↓
Document Findings
```

---

# 36. Evidence Directory

Recommended structure:

```text
database-lab/
├── evidence/
│   ├── original.sql
│   └── SHA256SUMS.txt
│
├── hashes/
│   ├── extracted.txt
│   └── normalized.txt
│
├── wordlists/
│   ├── small.txt
│   ├── numeric.txt
│   └── custom.txt
│
└── results/
    ├── john/
    └── hashcat/
```

Create:

```bash
mkdir -p database-lab/{evidence,hashes,wordlists,results/john,results/hashcat}
```

---

# 37. Preserve Evidence

Copy the original:

```bash
cp authorized-dump.sql database-lab/evidence/original.sql
```

Calculate:

```bash
sha256sum database-lab/evidence/original.sql
```

Save:

```bash
sha256sum database-lab/evidence/original.sql \
    > database-lab/evidence/SHA256SUMS.txt
```

Do not modify the original evidence file.

---

# 38. Keep Recovered Passwords Out of Git

Do not commit:

```text
*.pot
*.hash
*.sql
*.dump
passwords.txt
john.pot
hashcat.potfile
```

Add appropriate exclusions to `.gitignore`.

Example:

```gitignore
*.pot
*.potfile
*.hash
*.sql
*.dump
*.db
database-lab/evidence/
database-lab/results/
```

If the repository is public, never publish real credentials or password hashes from an unauthorized system.

---

# 39. Logging John

```bash
john \
    --wordlist=database-lab/wordlists/small.txt \
    database-lab/hashes/extracted.txt \
    2>&1 | tee database-lab/results/john/run.log
```

Show:

```bash
john --show database-lab/hashes/extracted.txt
```

---

# 40. Logging Hashcat

```bash
hashcat \
    -m <MODE> \
    database-lab/hashes/extracted.txt \
    database-lab/wordlists/small.txt \
    2>&1 | tee database-lab/results/hashcat/run.log
```

Show:

```bash
hashcat \
    -m <MODE> \
    database-lab/hashes/extracted.txt \
    --show
```

---

# 41. Sessions

Hashcat:

```bash
hashcat \
    --session=db-lab \
    -m <MODE> \
    database-lab/hashes/extracted.txt \
    database-lab/wordlists/small.txt
```

Status:

```bash
hashcat --session=db-lab --status
```

Resume:

```bash
hashcat --session=db-lab --restore
```

---

# 42. Compare John and Hashcat

Use identical:

```text
Hash set
Wordlist
Candidate space
Hardware
Attack strategy
```

Record:

```text
Tool
Version
Algorithm
Mode
Attack
Candidates
Speed
Runtime
Recovered
```

Example:

```text
Tool:       Hashcat
Algorithm:  SHA-256
Attack:     Dictionary
Candidates: 100000
Speed:      <measured>
Runtime:    <measured>
```

---

# 43. Multiple Hashes

If an authorized dump contains many hashes:

```bash
wc -l password-hashes.txt
```

Remove duplicates for benchmarking:

```bash
sort -u password-hashes.txt > unique-hashes.txt
```

Count:

```bash
wc -l unique-hashes.txt
```

Do not deduplicate if preserving the original evidence structure is important to your assessment.

Keep the original file untouched.

---

# 44. Salts and Duplicate Passwords

Two users may choose the same password.

With proper unique salts, their stored password representations should generally differ.

Laboratory experiment:

```text
User A:
password = labpassword
salt = saltA

User B:
password = labpassword
salt = saltB
```

Record:

```text
User
Password
Salt
Hash
Algorithm
```

This demonstrates why salts affect password-hash storage and recovery workflows.

---

# 45. Cost Parameters

For adaptive password-hashing schemes, inspect parameters such as:

```text
Cost
Iterations
Memory
Parallelism
Salt
Version
```

For example, a bcrypt-style value contains a cost parameter.

For Argon2-style values, parameters may include:

```text
Memory
Iterations
Parallelism
```

Do not treat these hashes like unsalted MD5 or SHA-256.

---

# 46. Application-Specific Formats

Some applications store passwords using custom formats:

```text
prefix$salt$hash
algorithm:salt:hash
iterations:salt:hash
```

Before cracking:

```text
1. Identify the application.
2. Identify its password-storage implementation.
3. Determine the exact format.
4. Extract the required fields.
5. Select the corresponding John/Hashcat format.
```

This is more reliable than guessing from appearance alone.

---

# 47. Troubleshooting

## Hashcat Mode Unknown

Search:

```bash
hashcat --example-hashes
```

Then:

```bash
hashcat --example-hashes | grep -Ei 'bcrypt|sha|md5|argon|pbkdf'
```

Verify the exact format.

---

## John Format Unknown

```bash
john --list=formats
```

Search:

```bash
john --list=formats | grep -Ei 'md5|sha|bcrypt|argon|pbkdf'
```

---

## Hashes Are Not Being Cracked

Check:

```text
Correct algorithm
Correct encoding
Correct salt
Correct cost
Correct separator
Correct mode
Correct candidate list
```

Do not change the hash data blindly to make it fit a tool.

---

## Password Not Found

Possible reasons:

```text
Password is not in the wordlist
Pattern is incorrect
Candidate space is too small
Wrong hash format
Wrong extraction
Wrong algorithm
Password is stronger than the tested space
```

Validate the complete workflow against a known laboratory password.

---

# 48. Complete Practical Workflow

```text
Authorized Database
        ↓
Create Evidence Copy
        ↓
Hash Evidence
        ↓
Identify User Table
        ↓
Identify Password Column
        ↓
Extract Password Hashes
        ↓
Identify Algorithm
        ↓
Identify Salt / Cost
        ↓
Validate Format
        ↓
John / Hashcat
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
Document
```

---

# 49. Quick Reference

### Install

```bash
sudo apt install john hashcat sqlite3
```

### SQLite

```bash
sqlite3 database.db ".tables"
sqlite3 database.db ".schema users"
```

### Extract

```bash
sqlite3 -noheader database.db \
    "SELECT username || ':' || password_hash FROM users;" \
    > hashes.txt
```

### Identify John Formats

```bash
john --list=formats
```

### Identify Hashcat Modes

```bash
hashcat --example-hashes
```

### John Dictionary

```bash
john \
    --wordlist=passwords.txt \
    hashes.txt
```

### John Rules

```bash
john \
    --wordlist=passwords.txt \
    --rules \
    hashes.txt
```

### John Result

```bash
john --show hashes.txt
```

### Hashcat Dictionary

```bash
hashcat \
    -m <MODE> \
    hashes.txt \
    passwords.txt
```

### Hashcat Mask

```bash
hashcat \
    -m <MODE> \
    hashes.txt \
    -a 3 \
    '?d?d?d?d'
```

### Hashcat Rules

```bash
hashcat \
    -m <MODE> \
    hashes.txt \
    passwords.txt \
    -r /usr/share/hashcat/rules/best64.rule
```

### Hashcat Result

```bash
hashcat \
    -m <MODE> \
    hashes.txt \
    --show
```

### Benchmark

```bash
john --test
hashcat -b
```

---

# 50. Research Checklist

```text
[ ] Database belongs to the laboratory or authorized assessment
[ ] Original evidence preserved
[ ] Evidence hash recorded
[ ] User/password table identified
[ ] Password column identified
[ ] Hashes extracted
[ ] Original extraction preserved
[ ] Algorithm identified
[ ] Salt identified
[ ] Cost parameters identified
[ ] John format verified
[ ] Hashcat mode verified
[ ] Small wordlist tested
[ ] Rules tested
[ ] Mask tested
[ ] Hybrid tested where appropriate
[ ] Results verified
[ ] Runtime recorded
[ ] Hardware recorded
[ ] Tool versions recorded
[ ] Real credentials excluded from Git
[ ] Database dumps excluded from Git
[ ] Cracking output excluded from Git
```

---

## Scope

Use this guide for:

* Your own databases
* Authorized penetration tests
* CTF environments
* Isolated password-security laboratories
* Defensive password-storage research
* Offline analysis of hashes obtained with explicit authorization

Do not use extracted credentials to access accounts or systems without authorization.
