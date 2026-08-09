# RainbowCrack — Practical Guide

> Practical RainbowCrack reference for authorized offline password-recovery research, CTFs, and isolated laboratories.

---

## 1. Install

### Kali Linux

First check whether the package is available:

```bash
sudo apt update
apt search rainbowcrack
```

If available:

```bash
sudo apt install rainbowcrack
```

Check:

```bash
rtcrack -h
```

Also check:

```bash
rcrack -h
```

Depending on the distribution/build, the installed binaries may differ.

---

# 2. Basic Components

RainbowCrack commonly uses:

```text
rtgen
rt2rtc
rtsort
rcrack
```

Basic workflow:

```text
Generate Tables
      ↓
Convert Tables
      ↓
Sort Tables
      ↓
Load Hashes
      ↓
Run Recovery
      ↓
Analyze Results
```

---

# 3. Help

Check each available binary:

```bash
rtgen -h
```

```bash
rt2rtc -h
```

```bash
rtsort -h
```

```bash
rcrack -h
```

If one command is unavailable:

```bash
which rtgen
which rt2rtc
which rtsort
which rcrack
```

---

# 4. Create Laboratory Directory

```bash
mkdir -p ~/rainbow-lab
cd ~/rainbow-lab
```

Create subdirectories:

```bash
mkdir -p tables
mkdir -p hashes
mkdir -p results
mkdir -p logs
```

Check:

```bash
tree ~/rainbow-lab
```

If `tree` is unavailable:

```bash
find ~/rainbow-lab -maxdepth 2 -type d
```

---

# 5. Create a Small Laboratory Password Dataset

Create:

```bash
cat > passwords.txt <<'EOF'
test
test1
test12
lab
lab1
lab123
password
password1
research
research123
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

This dataset is for controlled experiments.

---

# 6. Generate Laboratory Hashes

Use a hash utility to create hashes from your known test passwords.

For example, MD5:

```bash
printf '%s' 'lab123' | md5sum
```

SHA-1:

```bash
printf '%s' 'lab123' | sha1sum
```

SHA-256:

```bash
printf '%s' 'lab123' | sha256sum
```

Store only hashes generated specifically for your laboratory.

---

# 7. Create a Hash File

Example MD5 laboratory hash:

```bash
printf '%s' 'lab123' | md5sum | awk '{print $1}' > hashes/md5.txt
```

Check:

```bash
cat hashes/md5.txt
```

Verify:

```bash
wc -l hashes/md5.txt
```

---

# 8. Verify the Hash

Generate the hash again:

```bash
printf '%s' 'lab123' | md5sum
```

Compare it with:

```bash
cat hashes/md5.txt
```

The values should match.

---

# 9. Generate a Rainbow Table

RainbowCrack's `rtgen` syntax depends on the installed version.

Check:

```bash
rtgen -h
```

A typical table-generation structure is:

```text
rtgen hash_algorithm charset plaintext_length_min plaintext_length_max table_index chain_length chain_count
```

Example laboratory configuration:

```text
algorithm:
md5

charset:
loweralpha

minimum length:
1

maximum length:
4
```

Do not blindly copy parameters between versions; use the syntax shown by your local `rtgen -h`.

---

# 10. Example Table Generation

For a small laboratory experiment, inspect:

```bash
rtgen -h
```

Then construct the command using the supported syntax.

A typical configuration can look like:

```bash
rtgen md5 loweralpha 1 4 0 1000 100000
```

The exact parameters control:

```text
Hash algorithm
Character set
Minimum plaintext length
Maximum plaintext length
Table index
Chain length
Number of chains
```

Use small values first because table generation can consume substantial CPU, RAM, and disk resources.

---

# 11. Generate a Very Small Test Table

Start with a deliberately small experiment.

Example concept:

```text
Hash:
MD5

Charset:
loweralpha

Length:
1-3

Small chain parameters
```

Check the exact syntax:

```bash
rtgen -h
```

Then generate the table using parameters appropriate for your machine.

---

# 12. Check Generated Files

After `rtgen` completes:

```bash
ls -lah
```

Search for generated tables:

```bash
find . -type f
```

Move generated table files into:

```bash
mkdir -p tables
```

Keep generated files separated from your hash dataset.

---

# 13. Convert Tables

Some RainbowCrack workflows use `rt2rtc` to convert generated table files.

Check:

```bash
rt2rtc -h
```

Use the syntax displayed by your installed version.

General workflow:

```text
rtgen
  ↓
table files
  ↓
rt2rtc
  ↓
RTC table
```

---

# 14. Sort Tables

Use:

```bash
rtsort -h
```

RainbowCrack table files may need sorting before recovery.

Typical workflow:

```text
Generate
   ↓
Convert
   ↓
Sort
   ↓
Recover
```

Use the exact syntax provided by your version.

---

# 15. Move Tables

Keep tables organized:

```bash
mkdir -p tables/md5
```

Move the generated files:

```bash
mv *.rt tables/md5/ 2>/dev/null
```

Check:

```bash
ls -lah tables/md5
```

---

# 16. Inspect Table Files

```bash
file tables/md5/*
```

Check size:

```bash
du -sh tables/md5/
```

Large rainbow tables can consume significant disk space.

---

# 17. Prepare Hashes for Recovery

Create:

```bash
cat > hashes/test-md5.txt <<'EOF'
098f6bcd4621d373cade4e832627b4f6
EOF
```

This is the MD5 hash of the laboratory password:

```text
test
```

Verify:

```bash
printf '%s' 'test' | md5sum
```

---

# 18. Run RCRACK

Display help:

```bash
rcrack -h
```

General structure:

```text
rcrack <table-directory> -h <hash>
```

For a laboratory table directory:

```bash
rcrack tables/md5/ -h 098f6bcd4621d373cade4e832627b4f6
```

Use only tables that cover the selected hash algorithm and password space.

---

# 19. Recover From a Hash File

Some RainbowCrack versions support hash-file input.

Check:

```bash
rcrack -h
```

Look for options related to:

```text
hash file
hash list
```

Then provide:

```text
hashes/test-md5.txt
```

according to your installed version's syntax.

---

# 20. Multiple Laboratory Hashes

Create:

```bash
cat > hashes/lab-md5.txt <<'EOF'
098f6bcd4621d373cade4e832627b4f6
5f4dcc3b5aa765d61d8327deb882cf99
EOF
```

These correspond to known laboratory passwords.

Check:

```bash
cat hashes/lab-md5.txt
```

Count:

```bash
wc -l hashes/lab-md5.txt
```

---

# 21. Hash Recovery

Run:

```bash
rcrack -h
```

Determine the hash-file syntax for your installed version.

Then execute the corresponding recovery command against:

```text
tables/md5/
```

and:

```text
hashes/lab-md5.txt
```

---

# 22. Check Table Coverage

Before recovery, document:

```text
Algorithm
Charset
Minimum length
Maximum length
Chain length
Chain count
Table index
```

Example:

```text
Algorithm: MD5
Charset: lowercase
Length: 1-4
```

A hash outside the generated table's coverage will not be recovered by that table.

---

# 23. Character Sets

RainbowCrack tables depend heavily on the character set.

Common laboratory datasets can include:

```text
lowercase
uppercase
digits
lowercase + digits
uppercase + digits
mixed characters
```

Start with a small character set.

Example:

```text
abc
```

is much smaller than:

```text
abcdefghijklmnopqrstuvwxyz
```

---

# 24. Password-Length Coverage

Test separate ranges.

Example:

```text
1-3 characters
```

Then:

```text
1-4 characters
```

Then:

```text
1-5 characters
```

Record:

```text
Table size
Generation time
Recovery time
Recovery result
```

---

# 25. Table Indexes

Rainbow tables are commonly divided into different table indexes.

Example concept:

```text
Table 0
Table 1
Table 2
Table 3
```

Check your generated files:

```bash
find tables -type f
```

Use the table files corresponding to the desired configuration.

---

# 26. Chain Length

Chain length affects table generation and recovery characteristics.

Check:

```bash
rtgen -h
```

Start with small laboratory parameters.

Do not generate enormous tables on a production machine.

---

# 27. Chain Count

Chain count also affects table size.

For experiments:

```text
Small count
   ↓
Generate
   ↓
Measure
   ↓
Increase
   ↓
Compare
```

Record:

```text
Generation time
Disk usage
Recovery performance
Coverage
```

---

# 28. Benchmark Table Generation

Create an experiment:

```text
Table A
```

Record:

```text
Start time
End time
CPU usage
Disk size
```

Then create:

```text
Table B
```

Record the same values.

Compare:

```text
Parameter
Table A
Table B
```

---

# 29. Monitor CPU Usage

While generating tables:

```bash
top
```

or:

```bash
htop
```

If `htop` is unavailable:

```bash
sudo apt install htop
```

Monitor:

```text
CPU
RAM
Load
Disk usage
```

---

# 30. Monitor Disk Usage

Check:

```bash
df -h
```

Directory usage:

```bash
du -sh ~/rainbow-lab
```

Table usage:

```bash
du -sh ~/rainbow-lab/tables
```

Stop generation if your disk is approaching capacity.

---

# 31. Save Experiment Logs

Create:

```bash
mkdir -p logs
```

Record commands:

```bash
history | tail -n 30
```

Save notes:

```bash
nano logs/experiment-01.txt
```

Document:

```text
Machine
CPU
RAM
Algorithm
Charset
Length
Chain length
Chain count
Table size
Generation time
Recovery result
```

---

# 32. Result Logging

Create:

```bash
nano results/experiment-01.md
```

Example:

```text
# RainbowCrack Experiment 01

Algorithm: MD5
Charset: lowercase
Length: 1-4

Table:
[NAME]

Hash:
[LAB HASH]

Result:
Recovered / Not Recovered

Generation Time:
[TIME]

Recovery Time:
[TIME]
```

Never publish real password hashes.

---

# 33. Password-Recovery Experiment

Create known passwords:

```text
test
lab
abc
hello
```

Generate their hashes.

Create a table covering the corresponding search space.

Run:

```text
Hash
 ↓
Rainbow Table
 ↓
Candidate Lookup
 ↓
Recovered Password
```

Verify each result manually against the original laboratory dataset.

---

# 34. Failed Recovery

If recovery fails:

```text
Check algorithm
Check charset
Check password length
Check table files
Check table coverage
Check hash formatting
```

Inspect:

```bash
rcrack -h
```

A failed lookup does not necessarily mean the password cannot be recovered by another technique.

---

# 35. Hash Algorithm Experiment

Create separate laboratory datasets for:

```text
MD5
SHA-1
Other algorithms supported by your build
```

Check supported algorithms through:

```bash
rtgen -h
```

Do not assume every RainbowCrack build supports every modern hash algorithm.

---

# 36. Hash Validation

Before testing:

```bash
printf '%s' 'test' | md5sum
```

Expected:

```text
098f6bcd4621d373cade4e832627b4f6
```

Then:

```bash
cat hashes/test-md5.txt
```

Make sure the values match exactly.

---

# 37. File Formatting

Check for accidental whitespace:

```bash
cat -A hashes/test-md5.txt
```

Check line endings:

```bash
file hashes/test-md5.txt
```

Clean a simple hash list:

```bash
sed -i 's/\r$//' hashes/test-md5.txt
```

---

# 38. Hash List Cleanup

Remove blank lines:

```bash
sed -i '/^[[:space:]]*$/d' hashes/test-md5.txt
```

Check:

```bash
cat hashes/test-md5.txt
```

Count:

```bash
wc -l hashes/test-md5.txt
```

---

# 39. Organize Multiple Experiments

```bash
mkdir -p experiments/01-md5
mkdir -p experiments/02-md5-digits
mkdir -p experiments/03-md5-lowercase
```

Store:

```text
experiments/
├── 01-md5/
├── 02-md5-digits/
└── 03-md5-lowercase/
```

Keep each experiment independent.

---

# 40. Table Inventory

Create:

```bash
find tables -type f -printf '%p %s bytes\n'
```

Or:

```bash
du -ah tables | sort -h
```

This gives you a quick table inventory.

---

# 41. Recovery Workflow

```text
Create Known Laboratory Password
        ↓
Generate Authorized Hash
        ↓
Identify Algorithm
        ↓
Select Charset
        ↓
Select Password Length
        ↓
Generate Rainbow Table
        ↓
Convert Table
        ↓
Sort Table
        ↓
Prepare Hash File
        ↓
Run RCRACK
        ↓
Record Result
        ↓
Verify
```

---

# 42. Practical Benchmark

Run three table configurations:

```text
Configuration A
Small charset
Short passwords

Configuration B
Larger charset
Short passwords

Configuration C
Larger charset
Longer passwords
```

Record:

```text
Generation Time
Table Size
Recovery Time
Recovered Hashes
```

---

# 43. Compare Table Size

For each table:

```bash
du -sh tables/*
```

Create a report:

```text
Table       Size       Generation
----------------------------------
A           ...        ...
B           ...        ...
C           ...        ...
```

---

# 44. CPU Benchmark

Before generation:

```bash
lscpu
```

Record:

```text
CPU model
CPU cores
Threads
Architecture
```

During generation:

```bash
htop
```

Record approximate CPU utilization.

---

# 45. Memory Monitoring

```bash
free -h
```

During an experiment:

```bash
watch -n 1 free -h
```

Stop the experiment if system resources become unstable.

---

# 46. Storage Monitoring

```bash
watch -n 2 df -h
```

Table generation can consume substantial disk space.

Always maintain sufficient free storage.

---

# 47. Laboratory Cleanup

Remove temporary hashes:

```bash
rm -f hashes/tmp-*
```

Remove temporary tables:

```bash
rm -rf tables/tmp/
```

Check:

```bash
du -sh ~/rainbow-lab
```

If the VM is disposable, revert to the clean snapshot.

---

# 48. Troubleshooting

### `rtgen` Not Found

```bash
which rtgen
```

Check package:

```bash
apt search rainbowcrack
```

---

### `rcrack` Not Found

```bash
which rcrack
```

Check installed files:

```bash
dpkg -L rainbowcrack 2>/dev/null
```

---

### Table Generation Too Large

Reduce:

```text
Charset
Password length
Chain length
Chain count
```

Start with a very small test.

---

### Recovery Fails

Verify:

```bash
printf '%s' 'test' | md5sum
```

Compare:

```bash
cat hashes/test-md5.txt
```

Then check:

```text
Algorithm
Charset
Length
Table configuration
```

---

### No Table Files

Search:

```bash
find ~/rainbow-lab -type f
```

Check generation output.

Review:

```bash
rtgen -h
```

---

### Disk Full

Check:

```bash
df -h
```

Find large directories:

```bash
du -h ~/rainbow-lab | sort -h | tail
```

Remove unnecessary laboratory tables.

---

# 49. Practical Progression

```text
Level 1
Install
Help
Create Hashes
Basic RCRACK

        ↓

Level 2
RTGEN
Charsets
Password Length
Small Tables

        ↓

Level 3
RT2RTC
RTSORT
Table Organization
Hash Files

        ↓

Level 4
Multiple Hashes
Benchmarking
Table Comparison
Resource Monitoring

        ↓

Level 5
Large Laboratory Datasets
Coverage Analysis
Recovery Performance
Failure Analysis

        ↓

Level 6
Repeatable Research
Automated Experiments
Table Optimization
Offline Password-Auditing Labs
```

---

# 50. Quick Reference

```bash
# Check installation
which rtgen
which rt2rtc
which rtsort
which rcrack

# Help
rtgen -h
rt2rtc -h
rtsort -h
rcrack -h

# Create lab directory
mkdir -p ~/rainbow-lab
cd ~/rainbow-lab

# Create directories
mkdir -p tables hashes results logs

# Generate an MD5 laboratory hash
printf '%s' 'test' | md5sum

# Create a hash file
printf '%s' 'test' | md5sum | awk '{print $1}' > hashes/md5.txt

# Inspect hash
cat hashes/md5.txt

# Count hashes
wc -l hashes/md5.txt

# Check hash formatting
cat -A hashes/md5.txt

# Check CPU
lscpu

# Monitor memory
free -h

# Monitor disk
df -h

# Table inventory
find tables -type f

# Table size
du -sh tables

# Start recovery
rcrack -h

# Search for RainbowCrack files
find /usr -iname '*rainbow*' 2>/dev/null
```

---

# 51. Final Laboratory Workflow

```text
Authorized Password Dataset
        ↓
Generate Known Hashes
        ↓
Identify Hash Algorithm
        ↓
Select Search Space
        ↓
Generate Small Rainbow Table
        ↓
Convert / Sort Table
        ↓
Prepare Hash File
        ↓
Run RCRACK
        ↓
Record Recovery
        ↓
Verify Against Lab Dataset
        ↓
Benchmark
        ↓
Clean Up
        ↓
Document
```

---

# Scope

RainbowCrack is an offline password-recovery tool based on precomputed rainbow tables.

Use it only against:

* Your own password datasets
* Isolated laboratory hashes
* Authorized forensic datasets
* CTF environments
* Explicitly authorized security assessments

Do not use recovered hashes or password material from unauthorized systems or accounts.
