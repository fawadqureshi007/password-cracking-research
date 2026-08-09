# Password Manager Recovery

> Practical laboratory guide for password-manager vault recovery, offline passphrase testing, format identification, and security research.

---

## Scope

This guide covers:

```text
Password-manager vaults
Encrypted vault files
Vault format identification
Offline recovery
John the Ripper
Hashcat
Wordlists
Rules
Masks
Hybrid attacks
Key derivation
Recovery verification
Performance testing
```

Use only:

```text
Your own vault
Disposable laboratory vaults
CTF/lab environments
Explicitly authorized forensic evidence
```

Never attempt to recover credentials from someone else's password manager without authorization.

---

# 1. Lab Requirements

Recommended:

```text
Linux
Python 3
John the Ripper
Hashcat
KeePassXC
7-Zip
OpenSSL
```

Install common tools:

```bash
sudo apt update
sudo apt install john hashcat openssl python3 p7zip-full
```

Check:

```bash
john --version
hashcat --version
openssl version
python3 --version
```

Install a password manager separately if required by your distribution.

---

# 2. Create a Laboratory Vault

Use a disposable vault.

Example:

```text
Vault:
password-research-lab

Master password:
LabVault2026!
```

Add synthetic entries:

```text
Email:
lab@example.test

Username:
research-user

Password:
SyntheticPassword123!

Website:
https://example.test
```

Do not place real credentials in the laboratory vault.

---

# 3. Create a Backup

Before experimenting:

```bash
mkdir -p vault-lab/{original,working,hashes,wordlists,results}
```

Copy the laboratory vault:

```bash
cp laboratory-vault.kdbx \
    vault-lab/original/
```

Create a working copy:

```bash
cp laboratory-vault.kdbx \
    vault-lab/working/
```

Never modify the original evidence.

---

# 4. Hash the Original Vault

Calculate:

```bash
sha256sum \
    vault-lab/original/laboratory-vault.kdbx
```

Save:

```bash
sha256sum \
    vault-lab/original/laboratory-vault.kdbx \
    > vault-lab/original/SHA256SUMS.txt
```

Verify later:

```bash
sha256sum -c \
    vault-lab/original/SHA256SUMS.txt
```

---

# 5. Identify the Vault Format

Check:

```bash
file laboratory-vault.kdbx
```

For KeePass databases:

```text
.kdbx
```

The exact encryption and KDF parameters depend on the vault configuration.

Do not assume all password-manager formats use the same recovery method.

---

# 6. Inspect KeePass Metadata

For a laboratory KeePass database, inspect its configuration using the password-manager application.

Record:

```text
Vault format
Encryption algorithm
KDF
Iterations
Memory
Parallelism
Version
```

These parameters are important when comparing recovery performance.

---

# 7. KDF Types

Modern password managers may use:

```text
Argon2
AES-KDF
PBKDF2
scrypt
```

The purpose of a KDF is to make password guessing expensive.

Record:

```text
Algorithm
Iterations
Memory
Parallelism
Salt
```

Do not change the original laboratory vault while collecting these values.

---

# 8. Create a Laboratory Wordlist

```bash
cat > vault-passwords.txt <<'EOF'
password
password123
password1234
vault
vault123
vault2026
LabVault
LabVault2026
LabVault2026!
research
research123
EOF
```

Check:

```bash
wc -l vault-passwords.txt
```

---

# 9. John the Ripper

Check supported formats:

```bash
john --list=formats | grep -Ei 'keepass|kdbx|vault'
```

If your John installation includes the appropriate conversion utility, locate it:

```bash
find /usr -iname '*keepass*' 2>/dev/null
```

For supported KeePass databases, convert the laboratory vault into the format expected by John.

The exact helper name/path depends on your John installation.

---

# 10. John Dictionary Attack

After generating the appropriate John input:

```bash
john \
    --wordlist=vault-passwords.txt \
    vault.hash
```

Check:

```bash
john --show vault.hash
```

A successful result means a candidate matched the laboratory vault.

---

# 11. John Rules

```bash
john \
    --wordlist=vault-passwords.txt \
    --rules \
    vault.hash
```

Show:

```bash
john --show vault.hash
```

Rules are useful for testing common transformations:

```text
Uppercase
Lowercase
Capitalization
Numbers
Suffixes
Prefixes
Symbol additions
```

---

# 12. John Incremental Mode

Check:

```bash
john --list=incremental
```

Run only against the laboratory vault:

```bash
john \
    --incremental \
    vault.hash
```

Check:

```bash
john --status
```

Stop:

```text
Ctrl+C
```

---

# 13. John Sessions

Start:

```bash
john \
    --session=vault-lab \
    --wordlist=vault-passwords.txt \
    vault.hash
```

Status:

```bash
john \
    --status=vault-lab
```

Restore:

```bash
john \
    --restore=vault-lab
```

---

# 14. Hashcat

Check whether your installed Hashcat supports the vault format:

```bash
hashcat --example-hashes | grep -Ei 'keepass|kdbx'
```

Search the help output:

```bash
hashcat --help | grep -Ei 'keepass|kdbx'
```

Do not guess the mode number.

Use the exact mode supported by your installed version.

---

# 15. Hashcat Dictionary Attack

Once the correct mode is confirmed:

```bash
hashcat \
    -m <VAULT_MODE> \
    vault.hash \
    vault-passwords.txt
```

Show:

```bash
hashcat \
    -m <VAULT_MODE> \
    vault.hash \
    --show
```

Replace:

```text
<VAULT_MODE>
```

with the mode corresponding to the exact vault format.

---

# 16. Hashcat Rules

```bash
hashcat \
    -m <VAULT_MODE> \
    vault.hash \
    vault-passwords.txt \
    -r /usr/share/hashcat/rules/best64.rule
```

Show:

```bash
hashcat \
    -m <VAULT_MODE> \
    vault.hash \
    --show
```

---

# 17. Mask Attack

Suppose the laboratory master password follows:

```text
Vault + four digits
```

Run:

```bash
hashcat \
    -m <VAULT_MODE> \
    vault.hash \
    -a 3 \
    'Vault?d?d?d?d'
```

This is useful for controlled keyspace research.

---

# 18. Hybrid Attack

Word plus two digits:

```bash
hashcat \
    -m <VAULT_MODE> \
    vault.hash \
    -a 6 \
    vault-passwords.txt \
    '?d?d'
```

Two digits plus word:

```bash
hashcat \
    -m <VAULT_MODE> \
    vault.hash \
    -a 7 \
    '?d?d' \
    vault-passwords.txt
```

Only use these against laboratory vaults or explicitly authorized evidence.

---

# 19. Custom Character Set

Example six-character laboratory search:

```bash
hashcat \
    -m <VAULT_MODE> \
    vault.hash \
    -a 3 \
    -1 '?l?d' \
    '?1?1?1?1?1?1'
```

This tests lowercase letters and digits.

---

# 20. Known Pattern Testing

Laboratory password:

```text
LabVault2026
```

Known structure:

```text
LabVault + four digits
```

Test:

```bash
hashcat \
    -m <VAULT_MODE> \
    vault.hash \
    -a 3 \
    'LabVault?d?d?d?d'
```

This demonstrates the effect of predictable password construction.

---

# 21. Password-Length Experiment

Create multiple laboratory vaults:

```text
Vault A → short password
Vault B → medium password
Vault C → long password
Vault D → random password
Vault E → predictable password
```

Record:

```text
Length
Character set
KDF
Attack
Candidates
Speed
Runtime
Result
```

Compare the results.

---

# 22. KDF Performance Experiment

Create several disposable vaults using different KDF settings.

Record:

```text
KDF
Iterations
Memory
Parallelism
Hardware
Recovery speed
```

Example:

```text
Vault A → low cost
Vault B → medium cost
Vault C → high cost
```

The purpose is to measure how KDF configuration changes offline password-guessing cost.

---

# 23. Argon2 Research

Where the password manager supports Argon2, record:

```text
Argon2 variant
Memory
Iterations
Parallelism
Salt
```

Then benchmark the same candidate list against vaults using different parameters.

Do not extrapolate from one password manager to every Argon2 implementation.

---

# 24. PBKDF2 Research

For laboratory vaults using PBKDF2, record:

```text
Hash function
Iteration count
Salt
```

Compare:

```text
Low iterations
Medium iterations
High iterations
```

Use the same:

```text
Password
Hardware
Candidate list
```

for a meaningful comparison.

---

# 25. AES-KDF Research

If the vault uses AES-KDF, record:

```text
Cipher
Rounds
Transformations
Hardware
Recovery speed
```

Compare it against another laboratory configuration.

---

# 26. Recovery Verification

A recovered candidate should be verified by opening the disposable laboratory vault.

For example:

```text
Recovered:
LabVault2026!
```

Open the vault using the password-manager application.

Verify:

```text
Vault opens
Database contents are readable
Entries are intact
```

Do not verify recovered credentials against external services.

---

# 27. Export Test

After successfully opening the laboratory vault, export only synthetic test data.

Example:

```text
Username:
research-user

Password:
SyntheticPassword123!
```

Never export real credentials into the repository.

---

# 28. Vault Integrity

Before and after testing:

```bash
sha256sum \
    vault-lab/original/laboratory-vault.kdbx
```

Compare against:

```bash
sha256sum -c \
    vault-lab/original/SHA256SUMS.txt
```

The original should remain unchanged.

---

# 29. Benchmark John

```bash
john --test
```

Record:

```text
John version
CPU
GPU
Format
Speed
```

For meaningful research, run the same experiment under the same hardware conditions.

---

# 30. Benchmark Hashcat

```bash
hashcat -b
```

For a supported vault mode:

```bash
hashcat \
    -m <VAULT_MODE> \
    -b
```

Record:

```text
Hashcat version
GPU
Driver
Mode
Speed
```

---

# 31. John vs Hashcat

Use identical:

```text
Vault
Master password
Candidate list
Hardware
Attack type
```

Record:

```text
Tool
Format
Attack
Candidates
Speed
Runtime
Recovered
```

Example:

```text
Tool     | Attack     | Result
---------|------------|-------
John     | Dictionary | Measured
John     | Rules      | Measured
Hashcat  | Dictionary | Measured
Hashcat  | Mask       | Measured
```

Replace the example values with actual measurements.

---

# 32. Recovery Workflow

```text
Laboratory Vault
       ↓
Preserve Original
       ↓
Hash Evidence
       ↓
Identify Format
       ↓
Identify KDF
       ↓
Extract Supported Representation
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
Recovered Candidate
       ↓
Open Laboratory Vault
       ↓
Verify Contents
       ↓
Record Results
```

---

# 33. Evidence Structure

Recommended:

```text
password-manager-lab/
├── original/
│   └── laboratory-vault.kdbx
│
├── working/
│   └── laboratory-vault.kdbx
│
├── hashes/
│
├── wordlists/
│   └── vault-passwords.txt
│
└── results/
    ├── john.log
    ├── hashcat.log
    └── notes.md
```

Keep private vaults outside a public repository.

---

# 34. Git Protection

Add:

```gitignore
*.kdbx
*.kdb
*.vault
*.psafe3
*.json
*.hash
*.pot
*.potfile

password-manager-lab/original/
password-manager-lab/working/
password-manager-lab/results/
```

Be careful with broad patterns such as:

```text
*.json
```

because your project may legitimately contain JSON files.

A better approach is to ignore only directories containing sensitive material.

---

# 35. Never Commit

Do not commit:

```text
Real vault files
Real master passwords
Recovered passwords
API credentials
Cloud credentials
Browser exports
Password exports
Recovery codes
MFA secrets
Private keys
```

Use synthetic data for the public research repository.

---

# 36. Troubleshooting

## Tool Does Not Recognize the Vault

Check:

```text
Vault format
Password-manager version
Encryption
KDF
John version
Hashcat version
```

Do not assume that a `.kdbx` file always uses identical parameters.

---

## Password Not Found

Possible causes:

```text
Wrong extraction
Wrong vault format
Wrong tool mode
Incomplete candidate list
Incorrect attack pattern
Passphrase outside candidate space
Incorrect KDF parameters
```

First test with a deliberately simple laboratory password.

---

## Recovery Is Very Slow

Check:

```bash
john --test
```

and:

```bash
hashcat -b
```

Then inspect:

```text
CPU usage
GPU usage
Thermal throttling
KDF cost
Candidate generation
```

A deliberately expensive password KDF can make offline testing substantially slower.

---

# 37. Practical Experiment

Create a disposable vault with:

```text
Master password:
LabVault2026!
```

Create:

```bash
mkdir -p vault-research/{original,working,hashes,wordlists,results}
```

Create wordlist:

```bash
cat > vault-research/wordlists/test.txt <<'EOF'
password
password123
vault
vault123
LabVault
LabVault2026
LabVault2026!
EOF
```

Copy the vault:

```bash
cp laboratory-vault.kdbx \
    vault-research/original/
```

Create working copy:

```bash
cp laboratory-vault.kdbx \
    vault-research/working/
```

Calculate evidence hash:

```bash
sha256sum \
    vault-research/original/laboratory-vault.kdbx \
    > vault-research/original/SHA256SUMS.txt
```

Identify the vault:

```bash
file laboratory-vault.kdbx
```

Then use the appropriate password-manager-specific extraction method and run the recovered representation through John or Hashcat.

---

# 38. Advanced Experiment

Create several vaults:

```text
experiments/
├── low-kdf/
├── medium-kdf/
├── high-kdf/
├── predictable-password/
└── random-password/
```

For each one record:

```text
Vault
KDF
Parameters
Password length
Password structure
Attack
Candidate count
Hardware
Speed
Runtime
Recovery result
```

This produces measurable research data instead of a simple tool demonstration.

---

# 39. Research Questions

Use the laboratory to investigate:

```text
How does KDF selection affect recovery speed?
How does KDF cost affect candidate throughput?
How does password length affect recovery?
How does password predictability affect recovery?
How effective are wordlists?
How effective are rules?
How effective are masks?
How does GPU hardware change performance?
How do different vault formats differ?
```

Document actual measurements from your own experiments.

---

# 40. Results Template

```text
Experiment:
Date:
Password Manager:
Vault Format:
Encryption:
KDF:
Iterations:
Memory:
Parallelism:
Tool:
Tool Version:
Attack:
Wordlist:
Candidates:
Hardware:
Speed:
Runtime:
Recovered:
Verified:
```

---

# 41. Quick Reference

### Install

```bash
sudo apt install john hashcat openssl python3 p7zip-full
```

### John Formats

```bash
john --list=formats
```

### Search KeePass Support

```bash
john --list=formats | grep -Ei 'keepass|kdbx'
```

### Hashcat Vault Support

```bash
hashcat --example-hashes | grep -Ei 'keepass|kdbx'
```

### John Dictionary

```bash
john \
    --wordlist=vault-passwords.txt \
    vault.hash
```

### John Rules

```bash
john \
    --wordlist=vault-passwords.txt \
    --rules \
    vault.hash
```

### John Results

```bash
john --show vault.hash
```

### Hashcat Dictionary

```bash
hashcat \
    -m <VAULT_MODE> \
    vault.hash \
    vault-passwords.txt
```

### Hashcat Rules

```bash
hashcat \
    -m <VAULT_MODE> \
    vault.hash \
    vault-passwords.txt \
    -r /usr/share/hashcat/rules/best64.rule
```

### Hashcat Mask

```bash
hashcat \
    -m <VAULT_MODE> \
    vault.hash \
    -a 3 \
    'Vault?d?d?d?d'
```

### John Benchmark

```bash
john --test
```

### Hashcat Benchmark

```bash
hashcat -b
```

---

# 42. Research Checklist

```text
[ ] Disposable password-manager vault created
[ ] Synthetic credentials used
[ ] Original vault preserved
[ ] Evidence hash calculated
[ ] Vault format identified
[ ] Encryption identified
[ ] KDF identified
[ ] KDF parameters recorded
[ ] John support checked
[ ] Hashcat support checked
[ ] Dictionary attack tested
[ ] Rules tested
[ ] Mask tested
[ ] Hybrid tested
[ ] Benchmark completed
[ ] Recovery verified
[ ] Vault integrity verified
[ ] Real credentials excluded
[ ] Secrets excluded from Git
[ ] Results documented
```

---

## Safety

Use this research only with:

* Your own password-manager vaults
* Disposable laboratory vaults
* CTF environments
* Authorized forensic evidence
* Authorized security assessments

A password-manager vault can contain credentials for many systems. Treat the vault and any recovered master password as highly sensitive material.
