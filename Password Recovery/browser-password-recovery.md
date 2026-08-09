# Browser Password Recovery

> Practical laboratory guide for researching browser credential storage and recovering passwords from browser profiles you own or are explicitly authorized to examine.

---

## Scope

This guide covers:

```text
Browser profiles
Credential databases
SQLite inspection
Browser encryption
Profile identification
Forensic acquisition
Password recovery concepts
Offline analysis
Verification
Evidence handling
```

Supported research targets may include:

```text
Chromium-based browsers
Google Chrome
Microsoft Edge
Brave
Firefox
```

The exact storage format depends on the browser, operating system, and browser version.

---

# 1. Lab Requirements

Recommended:

```text
Linux VM
Python 3
SQLite3
Chromium/Chrome
Firefox
```

Install:

```bash
sudo apt update
sudo apt install sqlite3 python3
```

Check:

```bash
sqlite3 --version
python3 --version
```

---

# 2. Create a Dedicated Browser Lab

Do not use your normal browser profile.

Create a separate laboratory profile:

```text
browser-password-lab
```

Use only test accounts and disposable passwords.

Example:

```text
Username:
lab-user

Password:
LabPassword2026!
```

Do not use:

```text
Personal accounts
Email accounts
Banking accounts
Social-media accounts
Work accounts
Real credentials
```

---

# 3. Browser Profile Structure

A Chromium-style profile commonly contains files such as:

```text
Login Data
Cookies
Web Data
Preferences
History
```

Firefox uses a different profile structure.

Do not assume that a file named `Login Data` or `logins.json` contains plaintext passwords.

Modern browsers normally protect stored credentials using operating-system-backed encryption.

---

# 4. Locate Your Laboratory Profile

Chromium-based browser profiles vary by operating system and installation.

Search your lab directory rather than scanning unrelated user profiles:

```bash
find ~/browser-password-lab -type f
```

If you created a disposable Chromium profile manually:

```bash
find ~/browser-password-lab \
    -type f \
    | sort
```

Look for:

```text
Login Data
Local State
History
Cookies
Preferences
```

---

# 5. Preserve the Profile

Create an evidence directory:

```bash
mkdir -p browser-lab/evidence
```

Copy the disposable profile:

```bash
cp -a ~/browser-password-lab \
    browser-lab/evidence/profile
```

Calculate a checksum:

```bash
sha256sum -r browser-lab/evidence/profile \
    > browser-lab/evidence/profile-SHA256.txt
```

The original laboratory profile should remain untouched.

---

# 6. SQLite Databases

Chromium credential databases are commonly SQLite databases.

Identify:

```bash
file "Login Data"
```

Check:

```bash
sqlite3 "Login Data" ".tables"
```

Schema:

```bash
sqlite3 "Login Data" ".schema"
```

---

# 7. Inspect Login Records

For a laboratory copy:

```bash
sqlite3 "Login Data"
```

List tables:

```sql
.tables
```

Inspect the schema:

```sql
.schema logins
```

Exit:

```text
.quit
```

The schema varies between browser versions.

Do not assume column names from an old tutorial.

---

# 8. Query the Laboratory Database

For a compatible laboratory profile:

```bash
sqlite3 -header -column "Login Data" \
"SELECT origin_url, username_value, password_value FROM logins;"
```

You may see fields similar to:

```text
origin_url
username_value
password_value
```

The `password_value` field may contain encrypted data rather than plaintext.

---

# 9. Export Metadata

For forensic analysis, export non-secret metadata:

```bash
sqlite3 -header -csv "Login Data" \
"SELECT origin_url, username_value FROM logins;" \
> browser-logins.csv
```

Check:

```bash
cat browser-logins.csv
```

This provides a useful inventory without attempting to decrypt credentials.

---

# 10. Examine the Password Field

Inspect the laboratory database:

```bash
sqlite3 "Login Data" \
"SELECT length(password_value), hex(password_value) FROM logins;"
```

This allows you to examine the stored representation.

For example:

```text
length
hexadecimal representation
```

Do not assume the value is a crackable password hash.

---

# 11. Hash vs Encryption

Browser credential storage is fundamentally different from a traditional password-hash database.

Typical password-hash workflow:

```text
Password
   ↓
Hash function
   ↓
Stored hash
   ↓
Offline candidate testing
```

Browser credential storage commonly follows an encryption model:

```text
Password
   ↓
Encryption
   ↓
Encrypted credential
   ↓
Protected key material
```

Therefore:

```text
John the Ripper
Hashcat
```

are generally not substitutes for the browser's credential-decryption mechanism.

---

# 12. Chromium Encryption Architecture

Modern Chromium-based browsers can use:

```text
Browser profile
       ↓
Encrypted credential
       ↓
Browser encryption key
       ↓
Operating-system protection
```

Depending on the operating system, key protection may involve:

```text
Windows credential protection
macOS Keychain
Linux desktop secret storage
```

Exact implementation varies by browser and version.

---

# 13. Check the Browser Version

Record:

```bash
google-chrome --version
```

or:

```bash
chromium --version
```

For Edge:

```bash
microsoft-edge --version
```

For Brave:

```bash
brave-browser --version
```

Version information matters because browser storage formats change.

---

# 14. Browser Process Check

Before acquiring a live browser profile:

```bash
ps aux | grep -Ei 'chrome|chromium|firefox|brave|edge'
```

Close the laboratory browser before copying its database whenever possible.

This prevents:

```text
Locked databases
Incomplete writes
SQLite journal changes
Inconsistent evidence
```

---

# 15. SQLite Integrity

Check:

```bash
sqlite3 "Login Data" "PRAGMA integrity_check;"
```

Expected result for a healthy database:

```text
ok
```

Also check:

```bash
sqlite3 "Login Data" "PRAGMA journal_mode;"
```

---

# 16. Database Metadata

Inspect:

```bash
sqlite3 "Login Data" \
"SELECT name, type FROM sqlite_master ORDER BY type, name;"
```

This helps identify:

```text
Tables
Indexes
Views
Triggers
```

---

# 17. Firefox Profile

Firefox uses a different credential-storage architecture.

A Firefox profile may contain files such as:

```text
logins.json
key4.db
cookies.sqlite
places.sqlite
```

Locate only your disposable laboratory profile:

```bash
find ~/browser-password-lab \
    -type f \
    \( -name 'logins.json' -o -name 'key4.db' \)
```

---

# 18. Inspect `logins.json`

For a laboratory copy:

```bash
python3 -m json.tool logins.json
```

You may find records containing fields related to:

```text
hostname
encryptedUsername
encryptedPassword
```

The values are encrypted.

They should not be treated as plaintext password hashes.

---

# 19. Inspect Firefox Database

Check:

```bash
file key4.db
```

If it is SQLite:

```bash
sqlite3 key4.db ".tables"
```

Schema:

```bash
sqlite3 key4.db ".schema"
```

Do not modify the original evidence copy.

---

# 20. Firefox Encryption Model

A simplified model:

```text
Stored login
     ↓
Encrypted username
Encrypted password
     ↓
Encryption keys
     ↓
Protected browser profile
```

The encryption keys and browser profile work together.

Therefore, simply copying `logins.json` does not normally provide plaintext passwords.

---

# 21. Browser Forensics Workflow

```text
Disposable Browser Profile
          ↓
Close Browser
          ↓
Acquire Profile
          ↓
Hash Evidence
          ↓
Identify Browser + Version
          ↓
Identify Credential Database
          ↓
Inspect Schema
          ↓
Identify Encryption
          ↓
Determine Key Protection
          ↓
Authorized Recovery Procedure
          ↓
Verify Result
          ↓
Document Findings
```

---

# 22. Evidence Directory

Recommended structure:

```text
browser-lab/
├── evidence/
│   ├── chromium-profile/
│   ├── firefox-profile/
│   └── SHA256SUMS.txt
│
├── analysis/
│   ├── chromium/
│   └── firefox/
│
└── results/
    └── notes.md
```

Create:

```bash
mkdir -p browser-lab/{evidence,analysis,results}
```

---

# 23. Hash the Evidence

For individual files:

```bash
sha256sum "Login Data"
```

For a laboratory profile:

```bash
find browser-lab/evidence \
    -type f \
    -exec sha256sum {} \; \
    > browser-lab/evidence/SHA256SUMS.txt
```

Verify later:

```bash
sha256sum -c browser-lab/evidence/SHA256SUMS.txt
```

---

# 24. Metadata Collection

Record:

```text
Browser
Browser version
Operating system
Profile path
Acquisition time
Database files
File sizes
SHA-256 hashes
```

Example:

```text
Browser: Chromium
Version: <version>
OS: Linux
Profile: browser-password-lab
Database: Login Data
```

---

# 25. Compare Browser Versions

Create separate laboratory profiles:

```text
browser-lab/
├── chrome-old/
├── chrome-new/
├── firefox-old/
└── firefox-new/
```

Record:

```text
Version
Database schema
Encryption representation
Credential storage files
```

This is useful for researching how credential storage changes over time.

---

# 26. Credential-Storage Research

Create test accounts with controlled passwords:

```text
Account A:
Password = lab123

Account B:
Password = LabPassword123

Account C:
Password = Lab-Password-2026!
```

Store them only inside the disposable browser profile.

Then compare:

```text
Username
Origin
Encrypted credential length
Encrypted credential representation
Browser version
OS
```

Do not publish recovered passwords.

---

# 27. Database Schema Research

For Chromium:

```bash
sqlite3 "Login Data" ".schema logins"
```

Save the schema:

```bash
sqlite3 "Login Data" ".schema logins" \
    > browser-lab/analysis/chromium-login-schema.sql
```

For Firefox:

```bash
sqlite3 key4.db ".schema" \
    > browser-lab/analysis/firefox-key-schema.sql
```

---

# 28. SQLite Query Research

List all columns:

```bash
sqlite3 "Login Data" \
"PRAGMA table_info(logins);"
```

List indexes:

```bash
sqlite3 "Login Data" \
"PRAGMA index_list(logins);"
```

Count records:

```bash
sqlite3 "Login Data" \
"SELECT COUNT(*) FROM logins;"
```

---

# 29. Search Browser Artifacts

Within the disposable profile:

```bash
find browser-lab/evidence \
    -type f \
    | sort
```

Search filenames:

```bash
find browser-lab/evidence \
    -type f \
    | grep -Ei 'login|cookie|history|key|password|credential'
```

This gives an inventory of potentially relevant artifacts.

---

# 30. Cookies Are Different

Cookies are not password hashes.

Typical workflow:

```text
Browser
   ↓
Cookie database
   ↓
Encrypted cookie values
```

Do not send browser cookies into John or Hashcat expecting password recovery.

Cookies can represent authenticated sessions and therefore must be treated as sensitive authentication material.

---

# 31. Browser Password vs Password Hash

### Password database

```text
User
 ↓
Password
 ↓
Password hashing
 ↓
Stored hash
```

Tools such as:

```text
John
Hashcat
```

can test password candidates against compatible hash formats.

### Browser credential store

```text
Stored credential
 ↓
Encryption
 ↓
Protected key
 ↓
Operating-system security
```

The recovery problem is different.

---

# 32. Why Hashcat May Not Apply

If you run:

```bash
hashcat encrypted-browser-value
```

you cannot assume it will work.

The value may be:

```text
Encrypted credential
Not a password hash
Requires key material
Requires OS-backed protection
Browser-version dependent
```

Correctly identifying the storage mechanism is more important than choosing a cracking tool.

---

# 33. Offline Password-Recovery Research

If your research goal is specifically password cracking, use browser data only to study:

```text
Credential storage
Encryption
Key protection
Database structures
Forensic acquisition
```

For actual password-cracking experiments use:

```text
Extracted password hashes
Synthetic databases
Synthetic credential datasets
Known laboratory hashes
```

These are appropriate targets for:

```text
John the Ripper
Hashcat
Wordlists
Rules
Masks
Hybrid attacks
```

---

# 34. Create a Synthetic Browser Dataset

Create:

```text
browser-lab/
└── synthetic/
    ├── users.txt
    ├── hashes.txt
    └── metadata.csv
```

Example:

```text
users.txt
```

```text
alice
bob
charlie
```

Use deliberately generated hashes for experiments.

Keep:

```text
username
algorithm
salt
hash
password
```

documented separately.

---

# 35. Synthetic Hash Testing

Example:

```bash
printf 'labpassword' | sha256sum
```

Save:

```bash
printf 'labpassword' | sha256sum \
    | awk '{print $1}' \
    > synthetic.hash
```

Test with John:

```bash
john \
    --format=raw-sha256 \
    --wordlist=passwords.txt \
    synthetic.hash
```

Test with Hashcat:

```bash
hashcat \
    -m 1400 \
    synthetic.hash \
    passwords.txt
```

This keeps password-cracking experiments separate from browser credential extraction.

---

# 36. Recovery Verification

A laboratory recovery is successful only when:

```text
Recovered value
      ↓
Verified against known test credential
      ↓
Original laboratory account accepts it
```

Do not treat:

```text
Database record
Encrypted blob
Candidate string
```

as proof of successful password recovery.

---

# 37. Troubleshooting

## `Login Data` Is Locked

Close the laboratory browser:

```bash
ps aux | grep -Ei 'chrome|chromium|edge|brave'
```

Terminate only your laboratory browser process if necessary.

Then copy the database again.

---

## SQLite Reports Database Errors

Run:

```bash
sqlite3 "Login Data" "PRAGMA integrity_check;"
```

If the database is live, acquire it after closing the browser.

---

## `logins.json` Is Not Readable

Check:

```bash
file logins.json
```

Then:

```bash
python3 -m json.tool logins.json
```

---

## Password Field Looks Like Random Bytes

This is expected for encrypted credential storage.

Check:

```bash
sqlite3 "Login Data" \
"SELECT hex(password_value) FROM logins;"
```

Do not interpret the result as a normal password hash without identifying the browser's encryption format.

---

# 38. Research Comparison

Create a table:

```text
Browser | Version | OS | Credential Store | Encryption | Key Protection
```

Example:

```text
Chromium | <version> | Linux | Login Data | Encrypted | OS-backed
Firefox  | <version> | Linux | logins.json | Encrypted | Profile key
```

Fill in the actual values from your laboratory.

---

# 39. Recommended Lab Results

For each experiment record:

```text
Browser
Version
Operating System
Profile Type
Database
Database Schema
Encryption Format
Key Protection
Number of Credentials
Acquisition Method
Verification Result
```

---

# 40. Complete Practical Workflow

```text
Create Disposable Browser Profile
            ↓
Create Test Account
            ↓
Store Known Test Password
            ↓
Close Browser
            ↓
Acquire Profile
            ↓
Hash Evidence
            ↓
Identify Browser Version
            ↓
Locate Credential Database
            ↓
Inspect SQLite / JSON
            ↓
Identify Encryption
            ↓
Study Key Protection
            ↓
Recover Only Through Authorized Lab Methods
            ↓
Verify
            ↓
Document
```

---

# 41. Quick Reference

### Locate Laboratory Files

```bash
find ~/browser-password-lab -type f | sort
```

### SQLite Tables

```bash
sqlite3 "Login Data" ".tables"
```

### SQLite Schema

```bash
sqlite3 "Login Data" ".schema"
```

### Login Schema

```bash
sqlite3 "Login Data" \
"PRAGMA table_info(logins);"
```

### Login Count

```bash
sqlite3 "Login Data" \
"SELECT COUNT(*) FROM logins;"
```

### Login Metadata

```bash
sqlite3 -header -column "Login Data" \
"SELECT origin_url, username_value FROM logins;"
```

### Password Field Inspection

```bash
sqlite3 "Login Data" \
"SELECT length(password_value), hex(password_value) FROM logins;"
```

### SQLite Integrity

```bash
sqlite3 "Login Data" \
"PRAGMA integrity_check;"
```

### Firefox JSON

```bash
python3 -m json.tool logins.json
```

### Firefox Database

```bash
sqlite3 key4.db ".tables"
```

### Evidence Hash

```bash
sha256sum "Login Data"
```

---

# 42. Research Checklist

```text
[ ] Disposable browser profile created
[ ] Test account created
[ ] Test password used
[ ] Browser version recorded
[ ] Operating system recorded
[ ] Browser closed before acquisition
[ ] Profile copied
[ ] Evidence hashed
[ ] Credential database identified
[ ] SQLite schema inspected
[ ] Encryption identified
[ ] Key-protection mechanism documented
[ ] No personal credentials used
[ ] No real cookies used
[ ] No real session tokens used
[ ] Synthetic hashes used for cracking experiments
[ ] John/Hashcat used only against laboratory hashes
[ ] Recovery verified
[ ] Results documented
[ ] Sensitive artifacts excluded from Git
```

---

## Scope

Use this guide for:

* Your own browser profiles
* Disposable laboratory profiles
* Authorized digital-forensics examinations
* CTF environments
* Browser credential-storage research
* Password-security research

Do not use browser credential databases, cookies, session tokens, or encryption keys from another person's device or account without explicit authorization.
