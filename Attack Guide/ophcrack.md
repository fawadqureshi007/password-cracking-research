# Ophcrack — Practical Guide

> Practical Ophcrack reference for authorized Windows password-recovery labs, CTFs, and offline password-auditing research.

---

## 1. Install

### Kali Linux

```bash
sudo apt update
sudo apt install ophcrack
```

Check:

```bash
ophcrack --help
```

Version/package information:

```bash
apt show ophcrack
```

---

# 2. Basic Syntax

Ophcrack is primarily used for offline password recovery from Windows password hashes.

General structure:

```bash
ophcrack [options]
```

Display help:

```bash
ophcrack --help
```

---

# 3. GUI

Launch the graphical interface:

```bash
ophcrack
```

Typical workflow:

```text
Load Hashes
      ↓
Select Tables
      ↓
Start
      ↓
Analyze Results
```

Use only hashes obtained from systems/accounts you are authorized to test.

---

# 4. Check Installation

```bash
which ophcrack
```

Check package:

```bash
dpkg -l | grep ophcrack
```

Check version:

```bash
ophcrack --version
```

If the version option is unavailable:

```bash
ophcrack --help
```

---

# 5. Create an Isolated Windows Lab

Use a Windows VM dedicated to password-recovery research.

Example:

```text
Kali Linux
    |
    +---- Windows 10/11 VM
           |
           +---- labuser
           +---- Known laboratory password
```

Take a VM snapshot before testing.

---

# 6. Create a Laboratory Account

Inside the Windows laboratory VM, create a dedicated account:

```text
labuser
```

Use a password created specifically for the experiment.

For example:

```text
Lab12345
```

Do not use your personal Windows password.

---

# 7. Verify the Laboratory Account

Log into the Windows VM using:

```text
Username: labuser
Password: Lab12345
```

Confirm that the account works before extracting or importing any test hashes.

---

# 8. Hash Sources

Ophcrack works with Windows password hashes.

For a laboratory experiment, obtain hashes from:

```text
Your own Windows VM
Authorized forensic image
CTF challenge
Purpose-built password-recovery dataset
```

Do not extract password hashes from systems you do not own or have explicit permission to assess.

---

# 9. Import Hashes Using GUI

Start:

```bash
ophcrack
```

In the GUI:

```text
Load
   ↓
PWDUMP file
```

Select your authorized laboratory hash file.

After loading:

```text
Username
Hash
NT Hash
Status
```

will be displayed when applicable.

---

# 10. Hash File

A laboratory hash file may contain Windows account information in a format accepted by Ophcrack.

Example structure:

```text
labuser:RID:LM_HASH:NT_HASH:::
```

Use only hashes generated from your own laboratory environment.

Save the test file:

```bash
nano lab-hashes.txt
```

Then import it through Ophcrack.

---

# 11. Check the Hash File

Before loading:

```bash
cat lab-hashes.txt
```

Check the number of lines:

```bash
wc -l lab-hashes.txt
```

Inspect the file type:

```bash
file lab-hashes.txt
```

Never publish real NTLM hashes.

---

# 12. Windows Hash Types

Ophcrack is primarily associated with Windows LM/NTLM password recovery.

For modern Windows systems, NT hashes are generally more relevant than legacy LM hashes.

Identify your laboratory dataset before selecting an appropriate recovery method.

---

# 13. Tables

Ophcrack uses precomputed rainbow tables.

Check installed table-related packages:

```bash
apt search ophcrack
```

Search:

```bash
apt search rainbow
```

Available table packages depend on the distribution and repository configuration.

---

# 14. Check Table Directory

Inspect common locations:

```bash
find /usr/share -iname '*ophcrack*' 2>/dev/null
```

Also:

```bash
find /usr -iname '*table*' 2>/dev/null | grep -i ophcrack
```

Do not assume a table package is installed simply because Ophcrack is installed.

---

# 15. Table Selection

In the GUI:

```text
Tables
   ↓
Select
```

Choose tables appropriate for your laboratory hash type.

Then:

```text
Load
   ↓
Start
```

---

# 16. Start Recovery

After loading the hashes and tables:

```text
Start
```

Ophcrack will process the imported hashes against the selected tables.

Monitor:

```text
Progress
Hashes tested
Recovered passwords
Unsuccessful hashes
```

---

# 17. Stop a Recovery

If the experiment needs to be stopped:

```text
Stop
```

Record:

```text
Elapsed time
Tables used
Hashes tested
Recovered results
```

This makes the experiment reproducible.

---

# 18. Save Results

Use the GUI's result/export functionality to save the laboratory results.

Create an experiment directory:

```bash
mkdir -p experiments/ophcrack
```

Example:

```text
experiments/
└── ophcrack/
    ├── hashes/
    ├── tables/
    ├── results/
    └── notes/
```

Do not store real credentials in the repository.

---

# 19. Recovery Result

A successful laboratory result can be documented as:

```text
Account: labuser
Hash Type: NT
Recovery: Successful
Password: [LAB PASSWORD]
Table: [TABLE NAME]
Time: [TIME]
```

For public repositories, replace the recovered password with:

```text
[REDACTED]
```

---

# 20. Failed Recovery

If Ophcrack cannot recover a password:

```text
Status: Not recovered
```

Record:

```text
Hash type
Tables selected
Table size
Elapsed time
Hash count
```

Do not assume that an unrecovered password is necessarily strong; it may simply be outside the selected table coverage.

---

# 21. Verify a Laboratory Result

After recovering a password from your own Windows VM, manually verify it:

```text
Windows Login
      ↓
Username
      ↓
Recovered Laboratory Password
      ↓
Successful Authentication
```

Only perform this verification on the laboratory account.

---

# 22. Compare Passwords

Create several laboratory accounts:

```text
labweak
labmedium
labstrong
```

Give each account a different test password.

Example:

```text
labweak
123456

labmedium
Lab12345

labstrong
LongRandomLabPassword123!
```

Use these only for controlled research.

---

# 23. Compare Recovery Results

Record:

```text
Account
Password Pattern
Hash Type
Table Used
Recovered?
Time
```

Example:

```text
Account    Pattern       Result
--------------------------------
labweak    Numeric       Recovered
labmedium  Mixed         Recovered/Not recovered
labstrong  Long random  Not recovered
```

---

# 24. Benchmarking

Create a test dataset with known laboratory passwords.

Run:

```text
Test 01
Test 02
Test 03
Test 04
```

Record:

```text
Hash
Table
Start time
End time
Recovery status
```

This allows you to compare different table configurations.

---

# 25. Table Comparison

Test one table configuration:

```text
Table A
```

Record:

```text
Recovery:
Time:
Coverage:
```

Then test:

```text
Table B
```

Record the same fields.

Do not change multiple variables simultaneously if you want a meaningful benchmark.

---

# 26. Hash Count Testing

Create a laboratory dataset containing multiple hashes.

Example:

```text
5 hashes
10 hashes
25 hashes
50 hashes
```

Measure:

```text
Total processing time
Recovered hashes
Unrecovered hashes
Average recovery time
```

---

# 27. Password-Length Experiment

Create laboratory passwords with increasing lengths:

```text
Lab1
Lab12
Lab123
Lab1234
Lab12345
```

Generate their corresponding hashes inside your controlled environment.

Compare recovery results.

Document:

```text
Length
Character set
Hash type
Table
Recovery result
```

---

# 28. Character-Set Experiment

Create controlled passwords using:

```text
Lowercase
Uppercase
Numbers
Mixed characters
Symbols
```

Example:

```text
labtest
LabTest
LabTest123
LabTest123!
```

Compare whether the selected rainbow tables contain suitable coverage.

---

# 29. LM vs NTLM Laboratory Research

If your laboratory dataset contains both legacy LM and NT hashes, document them separately.

Example:

```text
Account: labuser

LM:
[LAB HASH]

NT:
[LAB HASH]
```

Do not publish actual reusable hashes from real systems.

---

# 30. Password Recovery Workflow

```text
Create Windows Lab
        ↓
Create Test Account
        ↓
Set Known Laboratory Password
        ↓
Generate/Obtain Authorized Hash
        ↓
Prepare Hash File
        ↓
Install Ophcrack
        ↓
Select Appropriate Tables
        ↓
Load Hashes
        ↓
Start Recovery
        ↓
Record Results
        ↓
Verify Laboratory Password
        ↓
Document Findings
```

---

# 31. Offline Research

Ophcrack performs offline recovery.

The basic model is:

```text
Windows Password
       ↓
Windows Password Hash
       ↓
Ophcrack
       ↓
Rainbow Tables
       ↓
Candidate Match
       ↓
Recovered Password
```

No live login attempts are required for the offline recovery stage.

---

# 32. Hash Security Experiment

Create a laboratory account with a known password.

Generate/obtain its authorized hash.

Run Ophcrack.

Then change the password to a longer random value.

Generate a new authorized test hash.

Run the same experiment.

Compare:

```text
Password
Length
Character Set
Table Coverage
Recovery
```

---

# 33. Result Documentation

Create:

```bash
nano experiments/ophcrack/results.md
```

Document:

```text
# Ophcrack Experiment

Target: Windows Laboratory VM
Account: labuser
Hash Type: NT
Tables: [TABLE]
Result: [RECOVERED / NOT RECOVERED]
Elapsed Time: [TIME]
```

Never commit real passwords or production hashes.

---

# 34. Hash File Protection

Restrict laboratory hash files:

```bash
chmod 600 lab-hashes.txt
```

Check:

```bash
ls -l lab-hashes.txt
```

For directories:

```bash
chmod 700 experiments/ophcrack
```

---

# 35. Secure Cleanup

After the experiment:

```bash
rm -f lab-hashes.txt
```

Remove temporary files:

```bash
rm -rf experiments/ophcrack/tmp
```

If the VM is disposable, revert to its clean snapshot.

---

# 36. Troubleshooting

### Command Not Found

```bash
which ophcrack
```

Install:

```bash
sudo apt update
sudo apt install ophcrack
```

---

### GUI Does Not Start

Check:

```bash
ophcrack --help
```

Then inspect:

```bash
echo $DISPLAY
```

If using a desktop Linux VM, verify that the graphical session is running.

---

### Hash File Not Loading

Check:

```bash
file lab-hashes.txt
```

Inspect:

```bash
cat lab-hashes.txt
```

Make sure the file format matches the input format expected by your installed Ophcrack version.

---

### No Tables Available

Search installed packages:

```bash
apt search ophcrack
```

Check:

```bash
find /usr/share -iname '*ophcrack*' 2>/dev/null
```

Install the appropriate table package for your distribution if available.

---

### Password Not Recovered

Check:

```text
Hash type
Table type
Table coverage
Password length
Character set
```

A failed recovery does not automatically mean the hash or password is invalid.

---

# 37. Laboratory VM Snapshot

Before changing the test environment:

```text
Windows VM
   ↓
Create Snapshot
   ↓
Password Experiment
   ↓
Hash Collection
   ↓
Ophcrack Recovery
   ↓
Results
   ↓
Revert Snapshot
```

This keeps experiments repeatable.

---

# 38. Practical Progression

```text
Level 1
Install
GUI
Help
Laboratory VM

        ↓

Level 2
Windows Hashes
Hash Files
Table Selection
Basic Recovery

        ↓

Level 3
Multiple Hashes
Multiple Accounts
Result Logging
Benchmarking

        ↓

Level 4
Password-Length Experiments
Character-Set Experiments
LM vs NT Research

        ↓

Level 5
Table Comparison
Recovery Benchmarking
Failure Analysis

        ↓

Level 6
Repeatable Labs
Forensic Datasets
Controlled Password Audits
Security Research
```

---

# 39. Quick Reference

```bash
# Install
sudo apt update
sudo apt install ophcrack

# Help
ophcrack --help

# Version
ophcrack --version

# Locate binary
which ophcrack

# Package information
apt show ophcrack

# Search Ophcrack packages
apt search ophcrack

# Search for installed Ophcrack files
find /usr/share -iname '*ophcrack*' 2>/dev/null

# Start GUI
ophcrack

# Inspect hash file
cat lab-hashes.txt

# Count hashes
wc -l lab-hashes.txt

# Identify file
file lab-hashes.txt

# Protect hash file
chmod 600 lab-hashes.txt

# Create experiment directory
mkdir -p experiments/ophcrack

# Protect experiment directory
chmod 700 experiments/ophcrack
```

---

# 40. Final Laboratory Workflow

```text
Authorized Windows VM
        ↓
Dedicated Test Account
        ↓
Known Laboratory Password
        ↓
Authorized Windows Hash
        ↓
Hash File
        ↓
Ophcrack
        ↓
Rainbow Tables
        ↓
Recovery Attempt
        ↓
Result
        ↓
Manual Verification
        ↓
Benchmark
        ↓
Document
        ↓
Clean Up
```

---

# Scope

Ophcrack is an offline password-recovery tool.

Use it only with:

* Your own Windows systems
* Authorized forensic images
* Isolated password-recovery laboratories
* CTF environments
* Explicitly authorized security assessments

Do not use password hashes obtained from unauthorized systems or accounts.
