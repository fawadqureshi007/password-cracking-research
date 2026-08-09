# SSH Key Recovery

> Practical laboratory guide for recovering passphrases from SSH private keys that you own or are explicitly authorized to examine.

---

## Scope

This guide covers:

```text
SSH private-key identification
OpenSSH key formats
Encrypted private keys
Passphrase recovery
John the Ripper
Hashcat
Wordlists
Rules
Mask attacks
Hybrid attacks
Key conversion
Benchmarking
Recovery verification
```

This guide does **not** cover stealing private keys, bypassing SSH authentication, or attacking SSH servers.

---

# 1. Lab Requirements

Recommended:

```text
Linux VM
OpenSSH
John the Ripper
Hashcat
Python 3
OpenSSL
```

Install:

```bash
sudo apt update
sudo apt install openssh-client openssl john hashcat python3
```

Verify:

```bash
ssh -V
openssl version
john --version
hashcat --version
```

---

# 2. Create a Laboratory Key

Generate an RSA test key:

```bash
mkdir -p ssh-key-lab
cd ssh-key-lab

ssh-keygen -t rsa -b 3072 \
    -f lab_rsa
```

When prompted for the passphrase, use a disposable laboratory value.

Example:

```text
LabSSH2026!
```

Files:

```text
lab_rsa
lab_rsa.pub
```

---

# 3. Generate an Ed25519 Test Key

```bash
ssh-keygen -t ed25519 \
    -f lab_ed25519
```

Use a disposable laboratory passphrase.

Check:

```bash
ls -l
```

---

# 4. Identify the Key Type

Run:

```bash
file lab_rsa
```

Inspect the beginning:

```bash
head -n 2 lab_rsa
```

Typical modern OpenSSH keys begin with:

```text
-----BEGIN OPENSSH PRIVATE KEY-----
```

Older PEM-style RSA keys may begin with:

```text
-----BEGIN RSA PRIVATE KEY-----
```

The format matters when selecting recovery tooling.

---

# 5. Check Public-Key Information

For RSA:

```bash
ssh-keygen -lf lab_rsa.pub
```

For Ed25519:

```bash
ssh-keygen -lf lab_ed25519.pub
```

This displays the public-key fingerprint.

---

# 6. Test the Passphrase

Attempt to read the private key:

```bash
ssh-keygen -y -f lab_rsa > /dev/null
```

Enter the laboratory passphrase.

Successful execution means the passphrase was accepted.

For Ed25519:

```bash
ssh-keygen -y -f lab_ed25519 > /dev/null
```

---

# 7. Verify the Public Key

Extract the public key:

```bash
ssh-keygen -y -f lab_rsa > recovered.pub
```

Compare:

```bash
ssh-keygen -lf lab_rsa.pub
ssh-keygen -lf recovered.pub
```

The fingerprints should match.

---

# 8. Inspect Private-Key Metadata

Run:

```bash
ssh-keygen -yf lab_rsa >/dev/null
```

The command prompts for the passphrase.

You can also inspect the file without modifying it:

```bash
head -n 3 lab_rsa
```

Do not publish private-key contents.

---

# 9. Create a Recovery Copy

Preserve the original:

```bash
mkdir -p evidence
cp lab_rsa evidence/lab_rsa.original
```

Hash it:

```bash
sha256sum evidence/lab_rsa.original \
    > evidence/SHA256SUMS.txt
```

Work from a copy:

```bash
cp evidence/lab_rsa.original lab_rsa.work
```

---

# 10. Why SSH Key Recovery Is Different

An SSH private key is not simply a password hash.

Conceptually:

```text
Private Key
     ↓
Encryption
     ↓
Encrypted Private Key
     ↓
Passphrase
```

Password recovery therefore involves testing candidate passphrases against the encrypted key.

The candidate must successfully decrypt or validate the key.

---

# 11. John the Ripper

John can process supported SSH private-key formats.

Check available formats:

```bash
john --list=formats | grep -i ssh
```

Create a John-compatible representation:

```bash
ssh2john lab_rsa > lab_rsa.hash
```

Depending on the installation, `ssh2john` may be located elsewhere.

Find it:

```bash
find /usr -name 'ssh2john*' 2>/dev/null
```

---

# 12. John Dictionary Attack

Create a laboratory wordlist:

```bash
cat > ssh-passwords.txt <<'EOF'
password
password123
ssh
ssh123
lab
lab123
LabSSH
LabSSH2026
LabSSH2026!
EOF
```

Run:

```bash
john \
    --wordlist=ssh-passwords.txt \
    lab_rsa.hash
```

Check:

```bash
john --show lab_rsa.hash
```

---

# 13. John Rules

Run:

```bash
john \
    --wordlist=ssh-passwords.txt \
    --rules \
    lab_rsa.hash
```

Show:

```bash
john --show lab_rsa.hash
```

Rules generate variations from the supplied laboratory wordlist.

---

# 14. John Incremental Mode

Check available modes:

```bash
john --list=incremental
```

Run against the laboratory key:

```bash
john \
    --incremental \
    lab_rsa.hash
```

Monitor:

```bash
john --status
```

Stop with:

```text
Ctrl+C
```

---

# 15. John Session

Start a named session:

```bash
john \
    --session=ssh-lab \
    --wordlist=ssh-passwords.txt \
    lab_rsa.hash
```

Check:

```bash
john \
    --status=ssh-lab
```

Restore:

```bash
john \
    --restore=ssh-lab
```

---

# 16. Show Recovered Passphrase

```bash
john --show lab_rsa.hash
```

A successful result identifies the recovered laboratory passphrase.

Do not publish real private-key passphrases.

---

# 17. Verify the Recovery

Use the recovered passphrase with:

```bash
ssh-keygen -y \
    -f lab_rsa
```

If the key is successfully decrypted, extract the public key:

```bash
ssh-keygen -y \
    -f lab_rsa \
    > recovered.pub
```

Compare fingerprints:

```bash
ssh-keygen -lf lab_rsa.pub
ssh-keygen -lf recovered.pub
```

Matching fingerprints confirm that the recovered passphrase unlocked the correct key.

---

# 18. Hashcat

Hashcat support depends on the exact SSH key format and installed version.

Inspect available SSH modes:

```bash
hashcat --example-hashes | grep -i ssh
```

List supported modes:

```bash
hashcat --help | grep -i ssh
```

Do not guess a mode number.

Use the mode corresponding to the exact extracted SSH format.

---

# 19. Hashcat Dictionary Attack

Once the correct Hashcat format has been identified:

```bash
hashcat \
    -m <SSH_MODE> \
    lab_rsa.hash \
    ssh-passwords.txt
```

Show recovered results:

```bash
hashcat \
    -m <SSH_MODE> \
    lab_rsa.hash \
    --show
```

Replace `<SSH_MODE>` with the mode supported by your installed version.

---

# 20. Hashcat Rules

```bash
hashcat \
    -m <SSH_MODE> \
    lab_rsa.hash \
    ssh-passwords.txt \
    -r /usr/share/hashcat/rules/best64.rule
```

Show:

```bash
hashcat \
    -m <SSH_MODE> \
    lab_rsa.hash \
    --show
```

---

# 21. Hashcat Mask Attack

If the laboratory passphrase follows a known pattern, test that pattern.

Example:

```text
SSH + four digits
```

Command:

```bash
hashcat \
    -m <SSH_MODE> \
    lab_rsa.hash \
    -a 3 \
    'SSH?d?d?d?d'
```

This is useful for controlled keyspace research.

---

# 22. Hybrid Attack

Word plus two digits:

```bash
hashcat \
    -m <SSH_MODE> \
    lab_rsa.hash \
    -a 6 \
    ssh-passwords.txt \
    '?d?d'
```

Two digits plus word:

```bash
hashcat \
    -m <SSH_MODE> \
    lab_rsa.hash \
    -a 7 \
    '?d?d' \
    ssh-passwords.txt
```

Only use these against laboratory keys or explicitly authorized material.

---

# 23. Custom Character Sets

Example:

```bash
hashcat \
    -m <SSH_MODE> \
    lab_rsa.hash \
    -a 3 \
    -1 '?l?d' \
    '?1?1?1?1?1?1'
```

This tests a controlled six-character lowercase/digit keyspace.

---

# 24. Known Prefix Research

Suppose a laboratory passphrase follows:

```text
LabSSH + 4 digits
```

Use:

```bash
hashcat \
    -m <SSH_MODE> \
    lab_rsa.hash \
    -a 3 \
    'LabSSH?d?d?d?d'
```

This demonstrates why predictable passphrase structures reduce the search space.

---

# 25. Password-Length Testing

Generate several laboratory keys:

```text
id="kqg0f4"
Key A → short passphrase
Key B → medium passphrase
Key C → long passphrase
Key D → random passphrase
```

Record:

```text
Key
Passphrase length
Character set
Attack
Candidates
Speed
Runtime
Result
```

Compare recovery difficulty.

---

# 26. Passphrase Complexity

Test controlled examples such as:

```text
ssh123
SSH123
LabSSH2026
Lab-SSH-2026
Lab-SSH-Key-2026!
```

The objective is to compare:

```text
Length
Predictability
Character diversity
Candidate-space size
Recovery time
```

Do not use real credentials for this experiment.

---

# 27. Generate Random Passphrases

For a laboratory experiment:

```bash
openssl rand -base64 24
```

Generate multiple:

```bash
for i in {1..5}; do
    openssl rand -base64 24
done
```

Use these only as disposable test passphrases.

---

# 28. Benchmark John

```bash
john --test
```

Record:

```text
John version
CPU
GPU
Algorithm
Speed
```

SSH key recovery speed depends heavily on the encryption format and hardware.

---

# 29. Benchmark Hashcat

```bash
hashcat -b
```

For the specific supported SSH mode:

```bash
hashcat \
    -m <SSH_MODE> \
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

# 30. Compare John and Hashcat

Use the same:

```text
SSH key
Passphrase
Candidate list
Hardware
```

Compare:

```text
Tool
Attack
Candidates
Speed
Runtime
Result
```

Example table:

```text
Tool     | Attack     | Result
---------|------------|-------
John     | Dictionary | Recovered
John     | Rules      | Recovered
Hashcat  | Dictionary | Recovered
Hashcat  | Mask       | Recovered
```

Replace these with your actual measurements.

---

# 31. RSA vs Ed25519

Create both:

```bash
ssh-keygen -t rsa -b 3072 -f rsa-lab
ssh-keygen -t ed25519 -f ed25519-lab
```

Record:

```text
Key type
Key size
File format
Encryption
Recovery-tool support
Performance
```

Do not assume every recovery tool supports every SSH format equally.

---

# 32. Legacy PEM Keys

Some older private keys use PEM encoding.

Example header:

```text
-----BEGIN RSA PRIVATE KEY-----
```

Modern OpenSSH keys commonly use:

```text
-----BEGIN OPENSSH PRIVATE KEY-----
```

Identify the format:

```bash
head -n 1 lab_rsa
```

Format identification should happen before selecting a recovery workflow.

---

# 33. Convert a Laboratory Key

If you need to study format differences, use a copy.

Example:

```bash
cp lab_rsa lab_rsa.converted
```

Then:

```bash
ssh-keygen \
    -p \
    -m PEM \
    -f lab_rsa.converted
```

The command will ask for the existing and new passphrases.

Verify:

```bash
head -n 1 lab_rsa.converted
```

Never convert the original evidence file.

---

# 34. Remove Passphrase From Lab Copy

For a disposable laboratory key only:

```bash
cp lab_rsa lab_rsa.no-passphrase
```

Then:

```bash
ssh-keygen \
    -p \
    -f lab_rsa.no-passphrase \
    -N ''
```

Verify:

```bash
ssh-keygen -y \
    -f lab_rsa.no-passphrase \
    > /dev/null
```

Do not do this to a real production private key.

---

# 35. SSH Agent

Check running agent:

```bash
ssh-add -l
```

Add the laboratory key:

```bash
ssh-add lab_rsa
```

List:

```bash
ssh-add -l
```

Remove:

```bash
ssh-add -d lab_rsa
```

Remove all keys from the laboratory agent:

```bash
ssh-add -D
```

Use a dedicated lab agent when experimenting with private keys.

---

# 36. Key Fingerprint Verification

Original:

```bash
ssh-keygen -lf lab_rsa.pub
```

Recovered:

```bash
ssh-keygen -y \
    -f lab_rsa \
    > recovered.pub

ssh-keygen -lf recovered.pub
```

Compare the fingerprints.

Expected:

```text
Original fingerprint
=
Recovered fingerprint
```

---

# 37. Evidence Structure

Recommended:

```text
ssh-key-lab/
├── evidence/
│   ├── original/
│   └── SHA256SUMS.txt
│
├── keys/
│   ├── laboratory/
│   └── converted/
│
├── hashes/
│
├── wordlists/
│
└── results/
    ├── john.log
    ├── hashcat.log
    └── notes.md
```

Keep private keys outside the public Git repository.

---

# 38. Hash Evidence

```bash
find evidence \
    -type f \
    -exec sha256sum {} \; \
    > evidence/SHA256SUMS.txt
```

Verify:

```bash
sha256sum -c evidence/SHA256SUMS.txt
```

---

# 39. Keep Secrets Out of Git

Add to `.gitignore`:

```gitignore
*.pem
*.key
*_rsa
*_ed25519
*.ppk
*.p12
*.pfx
*.hash
*.pot
*.potfile

ssh-key-lab/evidence/
ssh-key-lab/results/
```

Never commit:

```text
Private keys
Recovered passphrases
Real SSH credentials
Production keys
Known_hosts files containing sensitive infrastructure
```

---

# 40. Troubleshooting

## `ssh2john` Not Found

Search:

```bash
find /usr -name 'ssh2john*' 2>/dev/null
```

Common installations place John helper scripts outside the standard PATH.

Use the path appropriate to your installation.

---

## John Does Not Recognize the Hash

Check:

```bash
john --list=formats | grep -i ssh
```

Verify:

```text
Key format
John version
ssh2john output
```

Do not rename a hash or manually modify it to force a format.

---

## Hashcat Does Not Recognize the Format

Check:

```bash
hashcat --example-hashes | grep -i ssh
```

and:

```bash
hashcat --help | grep -i ssh
```

Hashcat support changes between releases.

Use the exact mode supported by your installed version.

---

## Passphrase Not Found

Possible causes:

```text
Wrong key file
Wrong extraction
Wrong format
Incomplete wordlist
Incorrect mask
Passphrase outside candidate space
Unsupported format
```

First validate the workflow with a known laboratory passphrase.

---

# 41. Recovery Verification Workflow

```text
Encrypted SSH Private Key
          ↓
Create Forensic Copy
          ↓
Identify Format
          ↓
Extract Supported Representation
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
Recovered Candidate
          ↓
Test With ssh-keygen
          ↓
Extract Public Key
          ↓
Compare Fingerprint
          ↓
Recovery Confirmed
```

---

# 42. Practical Experiment

Create a key:

```bash
ssh-keygen \
    -t rsa \
    -b 3072 \
    -f experiment-rsa
```

Use:

```text
LabSSH2026!
```

Create John input:

```bash
ssh2john experiment-rsa > experiment-rsa.hash
```

Create wordlist:

```bash
cat > experiment.txt <<'EOF'
password
password123
ssh123
LabSSH
LabSSH2026
LabSSH2026!
EOF
```

Run:

```bash
john \
    --wordlist=experiment.txt \
    experiment-rsa.hash
```

Show:

```bash
john --show experiment-rsa.hash
```

Verify:

```bash
ssh-keygen \
    -y \
    -f experiment-rsa \
    > recovered.pub
```

Compare:

```bash
ssh-keygen -lf experiment-rsa.pub
ssh-keygen -lf recovered.pub
```

---

# 43. Advanced Experiment

Create multiple keys:

```text
experiment/
├── short/
├── medium/
├── long/
├── random/
└── patterned/
```

Use different passphrase structures.

Run the same wordlist and attack methods against every key.

Record:

```text
Passphrase length
Pattern
Key format
Attack type
Candidate count
Recovery time
Result
```

This produces useful research data instead of simply demonstrating one successful crack.

---

# 44. Research Questions

Use the laboratory to investigate:

```text
Which SSH key formats are supported by John?
Which formats are supported by Hashcat?
How does passphrase length affect recovery?
How does predictability affect recovery?
How do rules affect candidate coverage?
How does GPU hardware change performance?
How does RSA key configuration affect processing?
How do modern OpenSSH formats differ from legacy formats?
```

Document measured results rather than assumptions.

---

# 45. Results Template

```text
Experiment:
Date:
Key Type:
Key Format:
Key Size:
Encryption:
Tool:
Tool Version:
Attack:
Wordlist:
Rules:
Mask:
Candidates:
Hardware:
Speed:
Runtime:
Recovered:
Verified:
Fingerprint Match:
```

---

# 46. Quick Reference

### Generate RSA Key

```bash
ssh-keygen -t rsa -b 3072 -f lab_rsa
```

### Generate Ed25519

```bash
ssh-keygen -t ed25519 -f lab_ed25519
```

### Fingerprint

```bash
ssh-keygen -lf lab_rsa.pub
```

### Extract Public Key

```bash
ssh-keygen -y -f lab_rsa > recovered.pub
```

### Find ssh2john

```bash
find /usr -name 'ssh2john*' 2>/dev/null
```

### Create John Input

```bash
ssh2john lab_rsa > lab_rsa.hash
```

### John Dictionary

```bash
john \
    --wordlist=ssh-passwords.txt \
    lab_rsa.hash
```

### John Rules

```bash
john \
    --wordlist=ssh-passwords.txt \
    --rules \
    lab_rsa.hash
```

### John Results

```bash
john --show lab_rsa.hash
```

### Hashcat SSH Modes

```bash
hashcat --example-hashes | grep -i ssh
```

### Hashcat

```bash
hashcat \
    -m <SSH_MODE> \
    lab_rsa.hash \
    ssh-passwords.txt
```

### Hashcat Results

```bash
hashcat \
    -m <SSH_MODE> \
    lab_rsa.hash \
    --show
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

# 47. Research Checklist

```text
[ ] Dedicated SSH laboratory created
[ ] Test RSA key generated
[ ] Test Ed25519 key generated
[ ] Disposable passphrases used
[ ] Original keys preserved
[ ] Evidence hashes calculated
[ ] Key format identified
[ ] Public fingerprint recorded
[ ] ssh2john tested
[ ] John format verified
[ ] Hashcat support checked
[ ] Dictionary attack tested
[ ] Rules tested
[ ] Mask tested
[ ] Hybrid tested
[ ] Benchmark completed
[ ] Recovered passphrase verified
[ ] Public-key fingerprint matched
[ ] Laboratory keys removed or rotated
[ ] Private keys excluded from Git
[ ] Real credentials excluded
[ ] Results documented
```

---

## Safety

Use this workflow only with:

* Your own SSH keys
* Disposable laboratory keys
* CTF/lab environments
* Explicitly authorized forensic examinations
* Authorized penetration-testing engagements

A recovered SSH private-key passphrase can provide access to systems when the corresponding key is authorized there. Treat private keys and recovered passphrases as sensitive credentials.
