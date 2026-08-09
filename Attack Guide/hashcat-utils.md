# Hashcat Utils — Practical Guide

> Practical Hashcat-utils reference for authorized offline password-auditing labs, CTFs, and password-security research.

---

# 1. Install

### Kali Linux

Check availability:

```bash
sudo apt update
apt search hashcat-utils
```

Install if available:

```bash
sudo apt install hashcat-utils
```

Check installed binaries:

```bash
dpkg -L hashcat-utils 2>/dev/null
```

---

# 2. Locate Utilities

```bash
which hcmask
which hcstat
which hcstat2
which maskprocessor
which combinator
which combinator3
which princeprocessor
```

If binaries are installed elsewhere:

```bash
find /usr -type f \( \
-name 'hcmask' \
-o -name 'hcstat*' \
-o -name 'combinator*' \
-o -name 'princeprocessor' \
-o -name 'maskprocessor' \
\) 2>/dev/null
```

---

# 3. Show Help

```bash
hcmask -h
```

```bash
hcstat2 -h
```

```bash
combinator -h
```

```bash
combinator3 -h
```

```bash
princeprocessor -h
```

```bash
maskprocessor -h
```

Use the help output from your installed version because available options can differ.

---

# 4. Create Laboratory Directory

```bash
mkdir -p ~/hashcat-utils-lab
cd ~/hashcat-utils-lab
```

Create directories:

```bash
mkdir -p input
mkdir -p output
mkdir -p masks
mkdir -p logs
mkdir -p results
```

Check:

```bash
find . -maxdepth 2 -type d
```

---

# 5. Create Test Wordlists

```bash
cat > input/words1.txt <<'EOF'
lab
test
admin
research
security
password
EOF
```

Second list:

```bash
cat > input/words2.txt <<'EOF'
123
1234
2026
lab123
test123
EOF
```

Check:

```bash
cat input/words1.txt
cat input/words2.txt
```

Count:

```bash
wc -l input/words1.txt
wc -l input/words2.txt
```

These are controlled laboratory candidates.

---

# 6. Combinator

`combinator` combines two candidate lists.

General structure:

```bash
combinator <left-list> <right-list>
```

Example:

```bash
combinator \
  input/words1.txt \
  input/words2.txt
```

Save output:

```bash
combinator \
  input/words1.txt \
  input/words2.txt \
  > output/combinations.txt
```

Check:

```bash
head output/combinations.txt
```

Count:

```bash
wc -l output/combinations.txt
```

---

# 7. Combinator Example

Input:

```text
lab
test
```

and:

```text
123
2026
```

Produces combinations conceptually similar to:

```text
lab123
lab2026
test123
test2026
```

This is useful for controlled password-pattern research.

---

# 8. Reverse Combination Order

Use the opposite list order:

```bash
combinator \
  input/words2.txt \
  input/words1.txt \
  > output/reverse-combinations.txt
```

Compare:

```bash
head output/reverse-combinations.txt
```

---

# 9. Combinator3

Check:

```bash
combinator3 -h
```

Use three controlled input sources where supported by your installed build.

Example workflow:

```text
List A
  +
List B
  +
List C
  ↓
Candidate Dataset
```

Always start with very small lists.

---

# 10. Estimate Candidate Growth

Before generating combinations:

```bash
wc -l input/words1.txt
wc -l input/words2.txt
```

If:

```text
List A = A candidates
List B = B candidates
```

then a simple two-list combination can produce approximately:

```text
A × B
```

candidates.

For three lists:

```text
A × B × C
```

This can grow extremely quickly.

---

# 11. PRINCE Processor

PRINCE generates candidate passwords from word material.

Check:

```bash
princeprocessor -h
```

Basic laboratory example:

```bash
princeprocessor < input/words1.txt
```

Save:

```bash
princeprocessor \
  < input/words1.txt \
  > output/prince.txt
```

Inspect:

```bash
head -n 50 output/prince.txt
```

---

# 12. PRINCE With Small Input

Create:

```bash
cat > input/prince-small.txt <<'EOF'
lab
red
team
test
EOF
```

Run:

```bash
princeprocessor \
  < input/prince-small.txt \
  > output/prince-small.txt
```

Inspect:

```bash
cat output/prince-small.txt
```

This makes it easier to understand candidate generation before using larger datasets.

---

# 13. Limit PRINCE Output

Check supported options:

```bash
princeprocessor -h
```

Look for options controlling:

```text
Minimum length
Maximum length
Maximum generated candidates
Input behavior
Output behavior
```

Use conservative limits during experiments.

---

# 14. Maskprocessor

Check:

```bash
maskprocessor -h
```

Maskprocessor generates structured candidate strings.

Typical character classes include:

```text
?l
```

Lowercase characters.

```text
?u
```

Uppercase characters.

```text
?d
```

Digits.

```text
?s
```

Special characters.

```text
?a
```

All supported character classes.

---

# 15. Simple Numeric Dataset

A four-digit laboratory candidate space can be generated with:

```bash
maskprocessor ?d?d?d?d
```

Save:

```bash
maskprocessor ?d?d?d?d \
  > output/four-digit.txt
```

Count:

```bash
wc -l output/four-digit.txt
```

This produces:

```text
0000
0001
0002
...
9999
```

---

# 16. Lowercase Mask

Example:

```bash
maskprocessor ?l?l?l
```

Save:

```bash
maskprocessor ?l?l?l \
  > output/lowercase-3.txt
```

Count:

```bash
wc -l output/lowercase-3.txt
```

---

# 17. Mixed Mask

Example:

```bash
maskprocessor ?l?l?d?d
```

Save:

```bash
maskprocessor ?l?l?d?d \
  > output/mixed-4.txt
```

Check:

```bash
head output/mixed-4.txt
```

---

# 18. Mask File

Create:

```bash
cat > masks/lab.hcmask <<'EOF'
?l?l?d?d
?l?l?l?d
?l?l?l?l
EOF
```

Inspect:

```bash
cat masks/lab.hcmask
```

Use the mask-file functionality supported by your installed `hcmask`/maskprocessor version.

---

# 19. HCMASK

Check:

```bash
hcmask -h
```

HCMASK files can describe structured mask combinations.

Inspect:

```bash
cat masks/lab.hcmask
```

Then test the file with the syntax documented by:

```bash
hcmask -h
```

Start with tiny masks before generating large candidate datasets.

---

# 20. Character Sets

Create a custom lowercase set:

```text
abcdef
```

Create:

```bash
cat > masks/custom.charset <<'EOF'
abcdef
EOF
```

Inspect:

```bash
cat masks/custom.charset
```

Use custom character sets only with the syntax supported by your installed version.

---

# 21. Candidate Deduplication

Generated datasets can contain duplicates.

Sort:

```bash
sort output/combinations.txt \
  > output/combinations-sorted.txt
```

Remove duplicates:

```bash
sort -u output/combinations.txt \
  > output/combinations-unique.txt
```

Count:

```bash
wc -l output/combinations.txt
wc -l output/combinations-unique.txt
```

---

# 22. Remove Empty Lines

```bash
sed -i '/^[[:space:]]*$/d' output/combinations-unique.txt
```

Verify:

```bash
grep -n '^$' output/combinations-unique.txt
```

No output means no empty lines were found.

---

# 23. Sort Candidates

```bash
sort output/prince-small.txt \
  > output/prince-small-sorted.txt
```

Check:

```bash
head output/prince-small-sorted.txt
```

---

# 24. Candidate Statistics

Count lines:

```bash
wc -l output/*.txt
```

Check file sizes:

```bash
du -h output/*
```

Check longest lines:

```bash
awk '{ if(length > max) max=length } END { print max }' \
  output/combinations.txt
```

---

# 25. Candidate Length Distribution

Use:

```bash
awk '{print length}' output/combinations.txt \
  | sort -n \
  | uniq -c
```

This helps analyze the generated candidate space.

---

# 26. Extract Specific Lengths

Candidates of exactly eight characters:

```bash
awk 'length($0)==8' \
  output/combinations.txt \
  > output/length-8.txt
```

Check:

```bash
wc -l output/length-8.txt
```

---

# 27. Extract a Length Range

Example:

```bash
awk 'length($0)>=6 && length($0)<=10' \
  output/combinations.txt \
  > output/length-6-to-10.txt
```

Check:

```bash
wc -l output/length-6-to-10.txt
```

---

# 28. Prefix Filtering

Keep candidates beginning with `lab`:

```bash
grep '^lab' \
  output/combinations.txt \
  > output/lab-prefix.txt
```

Check:

```bash
head output/lab-prefix.txt
```

---

# 29. Suffix Filtering

Keep candidates ending in `123`:

```bash
grep '123$' \
  output/combinations.txt \
  > output/123-suffix.txt
```

---

# 30. Numeric Filtering

Find candidates containing digits:

```bash
grep '[0-9]' \
  output/combinations.txt
```

Save:

```bash
grep '[0-9]' \
  output/combinations.txt \
  > output/contains-digits.txt
```

---

# 31. Alphabetic Filtering

Keep candidates containing only letters:

```bash
grep -E '^[A-Za-z]+$' \
  output/combinations.txt \
  > output/letters-only.txt
```

---

# 32. Prepare Candidates for Hashcat

After generating a laboratory candidate dataset:

```bash
sort -u output/combinations.txt \
  > output/final-candidates.txt
```

Check:

```bash
wc -l output/final-candidates.txt
```

Preview:

```bash
head output/final-candidates.txt
```

The resulting file can be used as an input wordlist for an authorized offline hash-auditing experiment.

---

# 33. Test Against a Laboratory Hash

Create a known test password:

```text
lab123
```

Generate its MD5 hash:

```bash
printf '%s' 'lab123' | md5sum
```

Create:

```bash
printf '%s' 'lab123' \
  | md5sum \
  | awk '{print $1}' \
  > hashes/test-md5.txt
```

---

# 34. Verify Candidate Dataset

Check whether the known password exists:

```bash
grep -Fx 'lab123' \
  output/final-candidates.txt
```

If it exists, the candidate-generation workflow is working.

---

# 35. Hashcat Integration

Check Hashcat:

```bash
hashcat --version
```

Help:

```bash
hashcat -h
```

Use your generated candidate file only with hashes created for your laboratory.

For example, the workflow is:

```text
Hashcat-utils
      ↓
Candidate Dataset
      ↓
Hashcat
      ↓
Authorized Laboratory Hash
      ↓
Result
```

---

# 36. Candidate Pipeline

Example:

```bash
combinator \
  input/words1.txt \
  input/words2.txt \
  > output/combinations.txt
```

Deduplicate:

```bash
sort -u \
  output/combinations.txt \
  > output/final-candidates.txt
```

Inspect:

```bash
head output/final-candidates.txt
```

Count:

```bash
wc -l output/final-candidates.txt
```

---

# 37. PRINCE Pipeline

```bash
princeprocessor \
  < input/words1.txt \
  > output/prince.txt
```

Deduplicate:

```bash
sort -u \
  output/prince.txt \
  > output/prince-unique.txt
```

Inspect:

```bash
head output/prince-unique.txt
```

---

# 38. Mask Pipeline

Generate:

```bash
maskprocessor ?l?l?d?d \
  > output/mask.txt
```

Deduplicate:

```bash
sort -u \
  output/mask.txt \
  > output/mask-unique.txt
```

Count:

```bash
wc -l output/mask-unique.txt
```

---

# 39. Combine Multiple Sources

Generate source A:

```bash
combinator \
  input/words1.txt \
  input/words2.txt \
  > output/source-a.txt
```

Generate source B:

```bash
maskprocessor ?d?d?d \
  > output/source-b.txt
```

Combine:

```bash
cat \
  output/source-a.txt \
  output/source-b.txt \
  > output/combined.txt
```

Remove duplicates:

```bash
sort -u \
  output/combined.txt \
  > output/combined-unique.txt
```

---

# 40. Split a Large Candidate File

Check:

```bash
wc -l output/combined-unique.txt
```

Split into chunks:

```bash
split -l 100000 \
  output/combined-unique.txt \
  output/chunk-
```

Check:

```bash
ls -lh output/chunk-*
```

---

# 41. Process a Candidate Chunk

Example:

```bash
wc -l output/chunk-aa
```

Preview:

```bash
head output/chunk-aa
```

Process each chunk separately in your laboratory workflow.

---

# 42. Parallel Candidate Generation

For independent experiments, generate separate datasets:

```text
Dataset A
Dataset B
Dataset C
```

Avoid generating several huge candidate spaces simultaneously unless your machine has sufficient CPU, RAM, and disk capacity.

Monitor:

```bash
htop
```

and:

```bash
df -h
```

---

# 43. Benchmark Combinator

Use:

```bash
time combinator \
  input/words1.txt \
  input/words2.txt \
  > output/benchmark-combinator.txt
```

Record:

```text
Input A size
Input B size
Output size
Generation time
Disk size
```

---

# 44. Benchmark PRINCE

```bash
time princeprocessor \
  < input/prince-small.txt \
  > output/benchmark-prince.txt
```

Record:

```bash
wc -l output/benchmark-prince.txt
du -h output/benchmark-prince.txt
```

---

# 45. Benchmark Mask Generation

```bash
time maskprocessor ?d?d?d?d \
  > output/benchmark-mask.txt
```

Count:

```bash
wc -l output/benchmark-mask.txt
```

Size:

```bash
du -h output/benchmark-mask.txt
```

---

# 46. Resource Monitoring

CPU:

```bash
htop
```

Memory:

```bash
free -h
```

Disk:

```bash
df -h
```

Directory size:

```bash
du -sh ~/hashcat-utils-lab
```

---

# 47. Reproducible Experiment

Create:

```bash
nano logs/experiment-01.txt
```

Record:

```text
Experiment:
Date:
OS:
CPU:
RAM:
Hashcat-utils version:
Input files:
Utility:
Parameters:
Candidate count:
Output size:
Generation time:
Notes:
```

This makes the experiment reproducible.

---

# 48. Hashcat-utils + Hashcat Workflow

```text
Input Wordlists
       ↓
Combinator / PRINCE
       ↓
Candidate Dataset
       ↓
Deduplicate
       ↓
Validate
       ↓
Hashcat
       ↓
Authorized Laboratory Hashes
       ↓
Record Result
```

---

# 49. Mask Workflow

```text
Define Password Pattern
       ↓
Select Character Classes
       ↓
Estimate Candidate Count
       ↓
Generate Small Test
       ↓
Validate Output
       ↓
Run Laboratory Audit
       ↓
Record Results
```

---

# 50. Candidate-Space Calculation

Before generating a mask, estimate its size.

For example, four decimal positions:

```text
10 × 10 × 10 × 10
```

equals:

```text
10,000
```

For six lowercase positions:

```text
26 × 26 × 26 × 26 × 26 × 26
```

equals:

```text
26^6
```

The search space grows rapidly as length and character-set size increase.

---

# 51. Avoid Accidental Huge Generation

Before running a mask:

```bash
maskprocessor -h
```

Calculate the expected candidate count first.

Start with:

```text
3 characters
```

Then:

```text
4 characters
```

Only increase the search space when the laboratory machine can safely handle the workload.

---

# 52. File Integrity

Create checksums:

```bash
sha256sum output/final-candidates.txt \
  > logs/final-candidates.sha256
```

Verify later:

```bash
sha256sum -c logs/final-candidates.sha256
```

---

# 53. Candidate Dataset Cleanup

Remove temporary files:

```bash
rm -f output/tmp-*
```

Remove duplicates:

```bash
sort -u \
  output/final-candidates.txt \
  > output/final-candidates.tmp
```

Replace:

```bash
mv output/final-candidates.tmp \
   output/final-candidates.txt
```

---

# 54. Search Candidate Dataset

Find a known test candidate:

```bash
grep -Fx 'lab123' \
  output/final-candidates.txt
```

Search for a prefix:

```bash
grep '^lab' \
  output/final-candidates.txt \
  | head
```

Search for a suffix:

```bash
grep '123$' \
  output/final-candidates.txt \
  | head
```

---

# 55. Candidate Length Statistics

```bash
awk '{print length}' \
  output/final-candidates.txt \
  | sort -n \
  | uniq -c
```

This gives a basic distribution of candidate lengths.

---

# 56. Candidate Dataset Report

Create:

```bash
nano results/dataset-report.md
```

Example:

```text
# Candidate Dataset Report

Input:
words1.txt
words2.txt

Utility:
combinator

Raw candidates:
[COUNT]

Unique candidates:
[COUNT]

Minimum length:
[VALUE]

Maximum length:
[VALUE]

Output size:
[SIZE]

Generation time:
[TIME]
```

---

# 57. Troubleshooting

### Command Not Found

```bash
which combinator
```

```bash
which princeprocessor
```

```bash
which maskprocessor
```

If unavailable:

```bash
apt search hashcat-utils
```

---

### Permission Denied

Check:

```bash
ls -l "$(which combinator)"
```

Do not blindly run unknown binaries as root.

---

### Output Is Huge

Stop generation with:

```text
CTRL+C
```

Then inspect:

```bash
du -h output/
df -h
```

Reduce the input lists or search-space size.

---

### Too Many Candidates

Check:

```bash
wc -l input/*.txt
```

For combinations, estimate:

```text
A × B
```

before generating.

---

### Duplicate Candidates

Use:

```bash
sort -u input.txt > unique.txt
```

---

### Unexpected Output

Check:

```bash
utility -h
```

Then verify the syntax for your installed version.

---

# 58. Practical Progression

```text
Level 1
Install
Help
Combinator
Small Wordlists

        ↓

Level 2
Combinator3
PRINCE
Maskprocessor
Candidate Statistics

        ↓

Level 3
Custom Masks
HCMASK
Deduplication
Filtering

        ↓

Level 4
Candidate Pipelines
Chunking
Benchmarking
Resource Monitoring

        ↓

Level 5
Hashcat Integration
Repeatable Experiments
Dataset Integrity
Performance Analysis

        ↓

Level 6
Large Authorized Datasets
Candidate-Space Optimization
Offline Password-Auditing Research
```

---

# 59. Quick Reference

```bash
# Search package
apt search hashcat-utils

# Locate utilities
which combinator
which combinator3
which princeprocessor
which maskprocessor
which hcmask

# Help
combinator -h
combinator3 -h
princeprocessor -h
maskprocessor -h
hcmask -h

# Create lab
mkdir -p ~/hashcat-utils-lab/{input,output,masks,logs,results}

# Two-list combination
combinator input/words1.txt input/words2.txt

# Save combination
combinator input/words1.txt input/words2.txt \
  > output/combinations.txt

# PRINCE
princeprocessor < input/words1.txt \
  > output/prince.txt

# Numeric four-character candidates
maskprocessor ?d?d?d?d \
  > output/four-digit.txt

# Lowercase three-character candidates
maskprocessor ?l?l?l \
  > output/lowercase-3.txt

# Deduplicate
sort -u output/combinations.txt \
  > output/combinations-unique.txt

# Count
wc -l output/combinations-unique.txt

# File size
du -h output/combinations-unique.txt

# Split
split -l 100000 output/combinations-unique.txt output/chunk-

# CPU
lscpu

# Memory
free -h

# Disk
df -h

# Checksum
sha256sum output/combinations-unique.txt
```

---

# 60. Final Laboratory Workflow

```text
Create Controlled Input
        ↓
Choose Utility
        ↓
Estimate Candidate Space
        ↓
Generate Small Dataset
        ↓
Inspect Output
        ↓
Remove Duplicates
        ↓
Filter / Split
        ↓
Validate Dataset
        ↓
Use With Authorized Offline Audit
        ↓
Benchmark
        ↓
Record Results
        ↓
Hash / Archive Research Data
        ↓
Clean Temporary Files
```

---

# Scope

Hashcat-utils provides utilities for preparing, transforming, and generating password candidates for offline password-security research.

Use these utilities only with:

* Your own password datasets
* Authorized password audits
* CTF environments
* Isolated laboratories
* Explicitly authorized security assessments

Do not use generated candidate datasets to attack third-party accounts or live authentication services.

Never publish real passwords, credential databases, or private password hashes.
