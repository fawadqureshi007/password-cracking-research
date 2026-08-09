# Password Cracking & Credential Recovery Research

A practical red-team research repository covering password cracking, credential recovery, encrypted-file attacks, authentication testing, and password security analysis.

The focus is on understanding how different password protection mechanisms can be assessed using tools such as **John the Ripper, Hashcat, Medusa, and Hydra** in controlled environments.

---

## Scope

This repository covers research and practical labs involving:

### Password Hashes

* MD5
* SHA family
* NTLM
* bcrypt
* scrypt
* PBKDF2
* Other supported password-hash formats

### Encrypted Files & Documents

* PDF password cracking
* ZIP password cracking
* RAR password cracking
* 7-Zip password cracking
* Microsoft Office password recovery
* SSH private-key password recovery
* KeePass database password recovery
* Other supported encrypted formats

### Authentication Testing

* Online password guessing
* Brute-force authentication testing
* Password spraying
* Credential validation
* Rate-limit testing
* Account-lockout testing

Online authentication techniques are covered only in controlled labs and authorized assessments.

---

## Tools

### John the Ripper

Research and labs covering:

* Hash identification
* Dictionary attacks
* Wordlist attacks
* Rules
* Incremental attacks
* Password recovery
* Encrypted-file formats
* Custom formats
* Performance testing

### Hashcat

Research and labs covering:

* Hash identification
* Dictionary attacks
* Rule-based attacks
* Mask attacks
* Hybrid attacks
* Brute-force techniques
* GPU acceleration
* Benchmarking
* Password-pattern research

### Medusa

Used for controlled research into online authentication security.

Topics include:

* Credential testing
* Authentication services
* Password guessing
* Rate limiting
* Account lockout
* Authentication defenses

### Hydra

Used in isolated authentication labs to study online credential attacks and defensive controls.

---

# Attack Techniques

The repository documents different password-recovery strategies and when they are useful.

## Dictionary Attacks

Testing candidate passwords from a wordlist.

Useful for passwords based on:

* Common passwords
* Dictionary words
* Names
* Dates
* Known password patterns
* Organization-specific terms

---

## Brute Force

Testing combinations from a defined character space.

The research covers how:

* Password length
* Character sets
* Search space
* Hardware
* Hash/KDF cost

affect recovery time.

---

## Rule-Based Attacks

Applying transformations to dictionary words.

Examples include:

```text
password
Password
password1
Password1
password!
P@ssword
P@ssword1
```

Research includes rule construction, optimization, and comparison with conventional dictionary attacks.

---

## Mask Attacks

Testing passwords when part of their structure is known.

Example:

```text
Word + 4 digits
```

Mask attacks are particularly useful when password-pattern information is available during an authorized assessment.

---

## Hybrid Attacks

Combining wordlists, masks, and mutations.

Examples:

```text
Word + Numbers
Word + Symbol
Word + Year
Mask + Word
Word + Mask
```

---

## Password Pattern Research

Research into predictable password construction, including:

* Names
* Years
* Seasons
* Company names
* Number sequences
* Common substitutions
* Repeated patterns
* Password mutations

---

# Encrypted File Research

A major part of the repository focuses on recovering passwords from protected files created specifically for laboratory research.

## PDF

Research includes:

* PDF protection mechanisms
* Password extraction
* Hash/verification representation
* Dictionary attacks
* Rule attacks
* Mask attacks
* Recovery analysis

## ZIP

Research includes:

* ZIP encryption
* Password extraction
* Dictionary attacks
* Rule attacks
* Mask attacks
* Recovery analysis

## RAR

Research into password-protected RAR archives and their supported recovery workflows.

## 7-Zip

Research into encrypted 7z archives, password verification, attack strategies, and performance.

## Microsoft Office

Research includes password-protected:

* DOCX
* XLSX
* PPTX

Different Office versions and protection mechanisms may behave differently and are documented separately.

## SSH Private Keys

Research into encrypted private-key passphrases and authorized password recovery.

## KeePass

Research into password-protected KeePass databases and the security properties of their key-derivation mechanisms.

---

# Authentication Attacks

Online authentication attacks are fundamentally different from offline password cracking.

This repository covers controlled research involving:

* SSH
* FTP
* HTTP authentication
* Other intentionally vulnerable lab services

Research areas include:

* Credential guessing
* Password spraying
* Rate limiting
* Account lockout
* Authentication monitoring
* Detection
* Defensive controls

No public or unauthorized authentication services are used.

---

# Research Methodology

Each technique follows a consistent workflow:

```text
Define Scope
     ↓
Identify Protection
     ↓
Analyze Target
     ↓
Select Attack Technique
     ↓
Prepare Test Data
     ↓
Run Controlled Test
     ↓
Measure Results
     ↓
Analyze Recovery
     ↓
Document Findings
     ↓
Evaluate Defenses
```

Each case study should document:

* Objective
* Target type
* Protection mechanism
* Tool used
* Attack technique
* Configuration
* Hardware
* Runtime
* Result
* Recovery status
* Analysis
* Limitations
* Defensive recommendations

---

# Repository Structure

```text
password-cracking-research/
│
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── SECURITY.md
│
├── 00-foundations/
│   ├── hashing-vs-encryption.md
│   ├── online-vs-offline.md
│   ├── salts.md
│   ├── kdf.md
│   ├── password-strength.md
│   └── cracking-time.md
│
├── 01-lab-setup/
│   ├── john.md
│   ├── hashcat.md
│   ├── medusa.md
│   ├── hydra.md
│   └── wordlists.md
│
├── 02-john-the-ripper/
│   ├── fundamentals/
│   ├── dictionary/
│   ├── rules/
│   ├── incremental/
│   ├── encrypted-files/
│   └── case-studies/
│
├── 03-hashcat/
│   ├── fundamentals/
│   ├── dictionary/
│   ├── rules/
│   ├── masks/
│   ├── hybrid/
│   ├── benchmarking/
│   └── case-studies/
│
├── 04-online-authentication/
│   ├── medusa/
│   ├── hydra/
│   ├── password-spraying/
│   ├── rate-limiting/
│   ├── account-lockout/
│   └── labs/
│
├── 05-encrypted-files/
│   ├── pdf/
│   ├── zip/
│   ├── rar/
│   ├── 7zip/
│   ├── office/
│   ├── ssh-keys/
│   └── keepass/
│
├── 06-attack-techniques/
│   ├── dictionary.md
│   ├── brute-force.md
│   ├── rules.md
│   ├── masks.md
│   ├── hybrid.md
│   ├── custom-wordlists.md
│   └── password-patterns.md
│
├── 07-credentials/
│   ├── linux/
│   ├── windows/
│   ├── applications/
│   └── lab-data/
│
├── 08-case-studies/
│   ├── weak-password/
│   ├── predictable-password/
│   ├── password-reuse/
│   ├── custom-wordlist/
│   ├── weak-kdf/
│   └── combined-techniques/
│
├── 09-benchmarking/
│   ├── cpu-vs-gpu.md
│   ├── hashcat/
│   ├── john/
│   └── comparisons/
│
├── 10-defense/
│   ├── password-storage.md
│   ├── password-policy.md
│   ├── kdf.md
│   ├── mfa.md
│   ├── rate-limiting.md
│   ├── account-lockout.md
│   └── detection.md
│
├── 11-labs/
│   ├── basic-hash/
│   ├── dictionary/
│   ├── rules/
│   ├── masks/
│   ├── pdf/
│   ├── zip/
│   ├── ssh/
│   └── full-assessment/
│
├── scripts/
│   ├── hash-identification/
│   ├── wordlist-generation/
│   ├── extraction/
│   └── benchmarking/
│
├── wordlists/
│   └── lab/
│
└── docs/
    ├── methodology.md
    ├── terminology.md
    ├── troubleshooting.md
    └── references.md
```

---

# Lab Environment

All practical research should be performed using:

* Self-generated hashes
* Self-created encrypted files
* Intentionally vulnerable machines
* CTF environments
* Authorized penetration-testing targets
* Synthetic credentials

A typical lab can consist of:

```text
Kali Linux
    │
    ├── John the Ripper
    ├── Hashcat
    ├── Medusa
    └── Hydra
          │
          ▼
   Isolated Test Network
          │
     ┌────┴────┐
     ▼         ▼
  Linux Lab  Windows Lab
```

---

# Performance Research

Password recovery performance depends on the algorithm, attack technique, candidate space, and hardware.

Benchmarks may compare:

```text
CPU vs GPU
John vs Hashcat
Dictionary vs Rules
Rules vs Masks
Fast Hash vs Slow KDF
```

Measurements can include:

* Candidates per second
* Runtime
* Hardware utilization
* Memory requirements
* Candidate count
* Recovery status

All benchmark results should include enough information to reproduce the experiment.

---

# Case Studies

The repository will contain complete controlled scenarios such as:

### Weak Password

Demonstrate how a weak password can be recovered using a suitable wordlist.

### Predictable Password

Study passwords constructed from predictable patterns such as:

```text
Word + Year
Name + Number
Company + Year
Word + Symbol
```

### Password Reuse

Demonstrate the security impact of reusing credentials across controlled lab systems.

### Custom Wordlist

Compare a generic wordlist against a context-specific laboratory wordlist.

### Weak KDF

Measure how an unsuitable password-derivation configuration affects offline recovery.

### Combined Techniques

Compare multiple attack strategies against the same synthetic target.

---

# Defensive Research

Every offensive technique should have a corresponding defensive section.

Research includes:

* Secure password storage
* Strong password policies
* Password managers
* MFA
* Salting
* Argon2id
* bcrypt
* scrypt
* PBKDF2
* Appropriate work factors
* Rate limiting
* Account lockout
* Authentication monitoring
* Credential-reuse detection

The objective is to understand both **how credentials can be recovered and how organizations can make recovery significantly harder**.

---

# Data Handling

Do not commit the following to this repository:

* Real passwords
* Stolen password databases
* Real user credentials
* Private keys belonging to others
* Session tokens
* API keys
* Corporate credential dumps
* Personal information
* Unauthorized access data

Use synthetic or intentionally created laboratory data instead.

---

# Roadmap

### Foundations

* [ ] Hashing vs encryption
* [ ] Online vs offline attacks
* [ ] Salts
* [ ] KDFs
* [ ] Password entropy
* [ ] Recovery-time estimation

### John the Ripper

* [ ] Installation
* [ ] Hash identification
* [ ] Dictionary attacks
* [ ] Rules
* [ ] Incremental attacks
* [ ] Encrypted files
* [ ] Case studies

### Hashcat

* [ ] Installation
* [ ] Hash identification
* [ ] Dictionary attacks
* [ ] Rules
* [ ] Masks
* [ ] Hybrid attacks
* [ ] Benchmarking
* [ ] Case studies

### Encrypted Files

* [ ] PDF
* [ ] ZIP
* [ ] RAR
* [ ] 7z
* [ ] DOCX
* [ ] XLSX
* [ ] PPTX
* [ ] SSH keys
* [ ] KeePass

### Authentication

* [ ] Medusa
* [ ] Hydra
* [ ] Password spraying
* [ ] Rate-limit testing
* [ ] Account-lockout testing
* [ ] Detection research

### Advanced Research

* [ ] Custom wordlists
* [ ] Password mutation
* [ ] Pattern analysis
* [ ] Hybrid techniques
* [ ] Combined attacks
* [ ] Performance comparisons

### Defense

* [ ] Password storage
* [ ] KDF configuration
* [ ] MFA
* [ ] Rate limiting
* [ ] Account lockout
* [ ] Monitoring
* [ ] Detection

---

# References

Primary documentation and established security references should be preferred:

* Hashcat documentation
* John the Ripper documentation
* Medusa documentation
* Hydra documentation
* OWASP Authentication Cheat Sheet
* OWASP Password Storage Cheat Sheet
* NIST Digital Identity Guidelines
* Cryptographic standards
* Academic password-security research

---

# Disclaimer

This repository is intended for **authorized security research, education, CTFs, penetration testing, and isolated laboratory environments**.

Do not use the techniques documented here against systems, accounts, files, or credentials without authorization.

The purpose of the project is to understand password security from a red-team perspective and use that knowledge to improve security.

---

## Research. Test. Understand. Secure.
