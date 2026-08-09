# Lab Setup

This section covers the environment used for controlled password-security research.

The objective is to build an isolated lab where password hashing, password recovery, authentication testing, and encrypted-file recovery can be tested without affecting real systems or accounts.

## Lab Objectives

The lab should provide:

* An isolated testing environment
* Linux-based security tools
* Synthetic password hashes
* Self-created encrypted files
* Test authentication services
* Repeatable experiments
* A consistent environment for benchmarking

All testing should be performed against systems and data that you own or have explicit authorization to assess.

---

## Recommended Environment

A practical setup can use:

```text
Host Machine
     │
     ├── Virtual Machine
     │      └── Kali Linux / Other Linux Distribution
     │
     ├── Test Machine
     │      └── Linux / Windows
     │
     └── Test Files
            ├── Hashes
            ├── PDF
            ├── ZIP
            ├── RAR
            ├── 7z
            └── Office Files
```

A virtualized environment makes it easier to create snapshots, reset experiments, and keep research data isolated.

---

# Operating System

Kali Linux is convenient for this type of research because many security-testing utilities are already available or can be installed through its package system.

Other Linux distributions can also be used.

The important requirement is not the distribution itself but having a reproducible environment with the required tools and dependencies.

---

# Core Tools

The research environment will focus on several categories of tools.

## John the Ripper

Used primarily for offline password-recovery research.

```text
John the Ripper
├── Password hashes
├── Dictionary attacks
├── Rules
├── Incremental attacks
└── Supported encrypted formats
```

See:

`../John-the-Ripper/`

---

## Hashcat

Hashcat is used for high-performance password-recovery research and supports a large number of hash and password-protection formats.

```text
Hashcat
├── Dictionary
├── Rules
├── Masks
├── Hybrid attacks
└── Benchmarking
```

See:

`../Hashcat/`

---

## Medusa

Medusa is intended for controlled online authentication testing against supported network services.

It should only be used against explicitly authorized targets.

See:

`../Online-Authentication/`

---

## Hydra

Hydra is another tool used for authorized online authentication testing.

Its use in this repository is limited to isolated laboratory environments and approved penetration-testing scenarios.

---

# Wordlists

Wordlists are collections of candidate passwords or words used during password-recovery testing.

The repository will distinguish between:

```text
Wordlists
├── Public wordlists
├── Synthetic lab wordlists
├── Custom research lists
└── Generated candidates
```

Do not commit credentials obtained from real users or unauthorized systems.

See:

`../Wordlists/`

---

# Creating Test Passwords

Use synthetic passwords for experiments.

Example categories:

```text
Weak
Predictable
Dictionary-based
Pattern-based
Random
Long passphrase
Random high-entropy password
```

The purpose is to test different password-generation characteristics rather than recover real credentials.

A research dataset can record:

```text
Password Type
Length
Character Set
Pattern
Generation Method
Expected Difficulty
```

The actual passwords do not need to be published when the experiment can be reproduced using a generator.

---

# Generating Hashes

For controlled experiments, create the password hashes yourself.

A basic workflow is:

```text
Synthetic Password
       ↓
Selected Hash / KDF
       ↓
Generated Hash
       ↓
Password-Recovery Test
       ↓
Record Result
```

Record the algorithm and relevant parameters for every experiment.

Example:

```text
Algorithm:       [document]
Salt:            [document]
Cost Parameters: [document]
Password Type:   [document]
```

---

# Encrypted-File Lab

Encrypted-file research should use files created specifically for the laboratory.

Example structure:

```text
Test Files
├── sample.pdf
├── sample.zip
├── sample.rar
├── sample.7z
├── sample.docx
├── sample.xlsx
└── sample.pptx
```

Each file should have a known test password and documented protection settings.

This makes recovery experiments reproducible.

---

# Authentication Lab

Online authentication testing should use dedicated test accounts and isolated services.

Example:

```text
Tester
  │
  ▼
Test Network
  │
  ├── SSH
  ├── FTP
  ├── HTTP Authentication
  └── Other Test Services
```

Do not use production accounts or public services for password-guessing experiments.

The lab should also allow defensive controls to be tested:

* Rate limiting
* Account lockout
* Authentication logging
* MFA
* Failed-login monitoring

---

# Network Isolation

Where possible, keep vulnerable laboratory services separated from the public Internet.

A basic arrangement is:

```text
Host
 │
 └── Virtual Network
       │
       ├── Attacker VM
       │
       └── Target VM
```

The attacker and target systems can communicate through the isolated virtual network without exposing the test services publicly.

---

# Snapshots

Virtual-machine snapshots are useful when experimenting with intentionally vulnerable configurations.

Before a major experiment:

```text
Clean Lab
   ↓
Create Snapshot
   ↓
Run Experiment
   ↓
Record Results
   ↓
Restore Snapshot
```

This allows the same experiment to be repeated from a known state.

---

# Hardware Documentation

Performance results should include the hardware used.

Record at least:

```text
CPU:
GPU:
RAM:
Operating System:
Driver Version:
John Version:
Hashcat Version:
```

Hardware can significantly affect offline password-recovery performance, especially during GPU-based testing.

---

# Research Directory

A consistent directory layout helps keep experiments reproducible.

Example:

```text
research/
├── hashes/
├── encrypted-files/
├── wordlists/
├── results/
├── benchmarks/
├── scripts/
└── notes/
```

Keep generated files and experimental output separate from the documentation.

---

# Experiment Documentation

Every experiment should record:

```text
Target
Protection / Algorithm
Parameters
Password Type
Candidate Source
Attack Technique
Tool
Tool Version
Hardware
Start Time
End Time
Result
Observations
```

Example:

```text
Target:       Synthetic test hash
Algorithm:    [document]
Tool:         [document]
Technique:    Dictionary + Rules
Candidates:   [document]
Hardware:     [document]
Result:       Recovered / Not recovered
Elapsed Time: [document]
```

This makes results easier to reproduce and compare.

---

# Safety

The laboratory should remain within an authorized scope.

Use:

* Self-generated credentials
* Self-created encrypted files
* CTF environments
* Intentionally vulnerable systems
* Explicitly authorized penetration-testing targets

Do not test password-guessing techniques against accounts, services, files, or credentials that you do not have permission to assess.

