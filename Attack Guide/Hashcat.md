# Hashcat — Practical Guide

> Practical Hashcat reference for authorized labs, CTFs, synthetic hashes, and self-created password-protected data.

---

## 1. Install

### Kali Linux

```bash
sudo apt update
sudo apt install hashcat
```

Check:

```bash
hashcat --version
```

Help:

```bash
hashcat --help
```

---

# 2. Check Hardware

List detected devices:

```bash
hashcat -I
```

Benchmark available devices:

```bash
hashcat -b
```

Benchmark a specific hash mode:

```bash
hashcat -b -m <mode>
```

Example:

```bash
hashcat -b -m 1000
```

---

# 3. Hash Modes

Hashcat identifies algorithms using numeric mode IDs.

List available modes:

```bash
hashcat --help
```

Search locally:

```bash
hashcat --help | grep -i ntlm
```

Examples:

```text
1000    NTLM
0       MD5
100     SHA1
1400    SHA-256
1700    SHA-512
3200    bcrypt
```

Always verify the mode with your installed Hashcat version.

---

# 4. Basic Hash File

Create a synthetic laboratory hash:

```bash
nano hash.txt
```

Put only your authorized test hash inside:

```text
<YOUR-LAB-HASH>
```

Test it:

```bash
hashcat -m <mode> hash.txt
```

---

# 5. Dictionary Attack

Basic:

```bash
hashcat -m <mode> hash.txt wordlist.txt
```

Example with a standard Kali wordlist:

```bash
hashcat \
  -m <mode> \
  hash.txt \
  /usr/share/wordlists/rockyou.txt
```

If compressed:

```bash
sudo gzip -dk /usr/share/wordlists/rockyou.txt.gz
```

Then:

```bash
hashcat \
  -m <mode> \
  hash.txt \
  /usr/share/wordlists/rockyou.txt
```

---

# 6. Show Results

Display recovered results:

```bash
hashcat -m <mode> hash.txt --show
```

Example:

```bash
hashcat -m 1000 hashes.txt --show
```

---

# 7. Save Recovered Results

Use an output file:

```bash
hashcat \
  -m <mode> \
  hash.txt \
  wordlist.txt \
  -o recovered.txt
```

Show:

```bash
cat recovered.txt
```

Never commit real recovered credentials to GitHub.

---

# 8. Restore / Session

Start a named session:

```bash
hashcat \
  --session=lab01 \
  -m <mode> \
  hash.txt \
  wordlist.txt
```

List sessions:

```bash
hashcat --session=lab01 --status
```

Resume:

```bash
hashcat --session=lab01 --restore
```

---

# 9. Wordlist + Rules

Basic wordlist:

```bash
hashcat -m <mode> hash.txt wordlist.txt
```

Apply a rule file:

```bash
hashcat \
  -m <mode> \
  hash.txt \
  wordlist.txt \
  -r rules/best64.rule
```

Common rule files can be found in the Hashcat installation directory.

Locate them:

```bash
find /usr/share -iname '*.rule' 2>/dev/null | head
```

---

# 10. Multiple Rule Files

Test another rule set:

```bash
hashcat \
  -m <mode> \
  hash.txt \
  wordlist.txt \
  -r rules/<rule-file>
```

For research, record:

```text
Wordlist
Rule File
Hash Mode
Hardware
Runtime
Result
```

---

# 11. Mask Attack

A mask attack generates candidates according to a defined structure.

Display built-in character sets:

```text
?l = lowercase
?u = uppercase
?d = digits
?s = special characters
?a = all supported character classes
```

Example synthetic target structure:

```text
4 lowercase characters + 2 digits
```

Command:

```bash
hashcat \
  -m <mode> \
  -a 3 \
  hash.txt \
  '?l?l?l?l?d?d'
```

---

# 12. Mask Examples

Six digits:

```bash
hashcat -m <mode> -a 3 hash.txt '?d?d?d?d?d?d'
```

Four lowercase letters:

```bash
hashcat -m <mode> -a 3 hash.txt '?l?l?l?l'
```

Four uppercase letters:

```bash
hashcat -m <mode> -a 3 hash.txt '?u?u?u?u'
```

Mixed:

```bash
hashcat -m <mode> -a 3 hash.txt '?u?l?l?l?d?d'
```

---

# 13. Custom Character Set

Define a custom character set:

```bash
hashcat \
  -m <mode> \
  -a 3 \
  -1 'abc123' \
  hash.txt \
  '?1?1?1?1'
```

Here `?1` represents the custom set.

Example:

```text
abc123
```

is used for every `?1` position.

---

# 14. Known Prefix

If a synthetic laboratory password has a known prefix:

```text
LAB + 4 digits
```

Use:

```bash
hashcat \
  -m <mode> \
  -a 3 \
  hash.txt \
  'LAB?d?d?d?d'
```

This is considerably smaller than searching every possible six-character combination.

---

# 15. Known Suffix

Example:

```text
4 digits + !
```

```bash
hashcat \
  -m <mode> \
  -a 3 \
  hash.txt \
  '?d?d?d?d!'
```

---

# 16. Known Structure

Example:

```text
SECURITY + 2 digits + !
```

```bash
hashcat \
  -m <mode> \
  -a 3 \
  hash.txt \
  'SECURITY?d?d!'
```

Use this only against a synthetic/authorized target where the structure is known.

---

# 17. Incrementing Mask Length

Hashcat can use mask files and mask-generation features for systematic research.

Check the installed options:

```bash
hashcat --help
```

For large experiments, define separate masks rather than immediately launching an unrestricted search.

Example:

```text
4 digits
5 digits
6 digits
```

Benchmark each separately.

---

# 18. Hybrid Attack — Word + Mask

Attack mode `6` combines a wordlist with a mask.

Example:

```bash
hashcat \
  -m <mode> \
  -a 6 \
  hash.txt \
  wordlist.txt \
  '?d?d'
```

Conceptually:

```text
word
word00
word01
word02
...
```

---

# 19. Hybrid Attack — Mask + Word

Attack mode `7` places the mask before the word.

```bash
hashcat \
  -m <mode> \
  -a 7 \
  hash.txt \
  '?d?d' \
  wordlist.txt
```

Conceptually:

```text
00word
01word
02word
...
```

---

# 20. Straight + Rules

Standard dictionary attack:

```bash
hashcat -m <mode> -a 0 hash.txt wordlist.txt
```

Dictionary + rules:

```bash
hashcat \
  -m <mode> \
  -a 0 \
  hash.txt \
  wordlist.txt \
  -r rules/best64.rule
```

---

# 21. Combination Attack

Attack mode `1` combines candidates from two wordlists.

```bash
hashcat \
  -m <mode> \
  -a 1 \
  hash.txt \
  first.txt \
  second.txt
```

For example:

```text
word1 + word2
```

This can be useful when testing synthetic passphrase-generation models.

---

# 22. Multiple Hashes

Put multiple laboratory hashes in one file:

```text
hash1
hash2
hash3
```

Run:

```bash
hashcat \
  -m <mode> \
  hashes.txt \
  wordlist.txt
```

Show:

```bash
hashcat -m <mode> hashes.txt --show
```

---

# 23. Username + Hash Formats

Some hash formats contain additional fields.

Example structure:

```text
username:hash
```

Do not manually remove fields unless the selected Hashcat mode expects that.

Check the required input format for the selected mode.

Useful command:

```bash
hashcat --example-hashes
```

Then search for the desired algorithm.

---

# 24. Example Hashes

Hashcat can show example input formats:

```bash
hashcat --example-hashes
```

Search:

```bash
hashcat --example-hashes | grep -i ntlm
```

This is useful when building synthetic laboratory targets.

---

# 25. Hash Identification

Hashcat does not always automatically identify an unknown hash.

Useful information includes:

```text
Length
Prefix
Salt
Separators
Encoding
Known Application
```

Then search the supported modes:

```bash
hashcat --help
```

For example:

```bash
hashcat --help | grep -i bcrypt
```

---

# 26. Salted Hashes

Some modes expect the salt as part of the input.

Do not guess the format.

Use:

```bash
hashcat --example-hashes
```

and compare your synthetic target with the documented input structure.

Then:

```bash
hashcat -m <mode> hash.txt wordlist.txt
```

---

# 27. Performance Information

During a running attack, Hashcat displays information such as:

```text
Speed
Progress
Recovered
Candidates
Time
Hardware Utilization
```

Use:

```text
s
```

inside the interactive Hashcat session to display status.

---

# 28. Benchmarking

Full benchmark:

```bash
hashcat -b
```

Specific mode:

```bash
hashcat -b -m <mode>
```

Benchmark with workload settings where supported:

```bash
hashcat -b -m <mode> -w 3
```

Record:

```text
Hash Mode
Device
Speed
Runtime
Driver
Hashcat Version
```

---

# 29. Workload Profiles

Hashcat supports workload profiles.

Example:

```bash
hashcat -m <mode> -a 0 hash.txt wordlist.txt -w 3
```

Higher workload profiles can increase resource utilization.

Use them carefully on a workstation because they can make the system less responsive.

---

# 30. Device Selection

List devices:

```bash
hashcat -I
```

Run against a specific device:

```bash
hashcat \
  -d 1 \
  -m <mode> \
  hash.txt \
  wordlist.txt
```

Multiple devices:

```bash
hashcat \
  -d 1,2 \
  -m <mode> \
  hash.txt \
  wordlist.txt
```

Use the device IDs reported by your system.

---

# 31. CPU Testing

For controlled benchmarking, select the CPU device reported by:

```bash
hashcat -I
```

Then run:

```bash
hashcat \
  -d <cpu-device-id> \
  -m <mode> \
  hash.txt \
  wordlist.txt
```

Compare the result with the GPU configuration.

Keep the target and candidate set identical.

---

# 32. GPU Testing

Check:

```bash
hashcat -I
```

Then select the GPU:

```bash
hashcat \
  -d <gpu-device-id> \
  -m <mode> \
  hash.txt \
  wordlist.txt
```

Benchmark:

```bash
hashcat -b -d <gpu-device-id>
```

---

# 33. Optimized Kernels

Hashcat may report whether an optimized kernel is available for a selected mode.

Inspect the startup output.

For a controlled experiment, compare:

```bash
hashcat \
  -m <mode> \
  -a 0 \
  hash.txt \
  wordlist.txt
```

with the appropriate optimized configuration supported by your installed version.

Do not enable options blindly. Read the warning/output because optimized kernels can impose password-length or feature limitations.

---

# 34. Restore Interrupted Work

Start:

```bash
hashcat \
  --session=experiment01 \
  -m <mode> \
  hash.txt \
  wordlist.txt
```

Interrupt safely:

```text
q
```

Restore:

```bash
hashcat --session=experiment01 --restore
```

---

# 35. Potfile

Hashcat stores recovered results in its potfile.

Show recovered passwords:

```bash
hashcat -m <mode> hash.txt --show
```

Find configuration information:

```bash
hashcat -I
```

Do not publish the potfile.

Add it to `.gitignore`:

```bash
echo "*.potfile" >> .gitignore
```

---

# 36. Remove Recovered Hashes

For a fresh laboratory experiment, create a new target file containing only unrecovered hashes.

You can inspect the current results with:

```bash
hashcat -m <mode> hash.txt --show
```

Keep separate experiment directories so previous results do not contaminate new benchmarks.

---

# 37. Wordlist Statistics

Count candidates:

```bash
wc -l wordlist.txt
```

Remove duplicates:

```bash
sort -u wordlist.txt > clean.txt
```

Count again:

```bash
wc -l clean.txt
```

Inspect:

```bash
head clean.txt
tail clean.txt
```

---

# 38. Generate Synthetic Candidates

Example:

```bash
for i in $(seq -w 0 9999); do
    echo "LAB$i"
done > lab.txt
```

Count:

```bash
wc -l lab.txt
```

Run:

```bash
hashcat \
  -m <mode> \
  -a 0 \
  hash.txt \
  lab.txt
```

---

# 39. Python Candidate Generator

For a synthetic research pattern:

```python
with open("candidates.txt", "w") as f:
    for year in range(2020, 2031):
        for number in range(100):
            f.write(f"Research{year}{number:02d}!\n")
```

Then:

```bash
hashcat \
  -m <mode> \
  -a 0 \
  hash.txt \
  candidates.txt
```

Record:

```bash
wc -l candidates.txt
```

---

# 40. Rule Testing

Test one rule file:

```bash
hashcat \
  -m <mode> \
  -a 0 \
  hash.txt \
  wordlist.txt \
  -r rules/best64.rule
```

Test another:

```bash
hashcat \
  -m <mode> \
  -a 0 \
  hash.txt \
  wordlist.txt \
  -r rules/<another-rule>.rule
```

Compare:

```text
Candidate Count
Speed
Runtime
Recovery
```

---

# 41. Rule Combination

For research, multiple rule files can be tested independently.

```bash
for rule in rules/*.rule; do
    echo "[+] Testing $rule"

    hashcat \
      -m <mode> \
      -a 0 \
      hash.txt \
      wordlist.txt \
      -r "$rule"
done
```

Use this only with synthetic targets and controlled wordlists.

---

# 42. Mask Benchmarking

Create several laboratory masks:

```text
?d?d?d?d
?d?d?d?d?d
?d?d?d?d?d?d
```

Benchmark them individually:

```bash
hashcat -m <mode> -a 3 hash.txt '?d?d?d?d'
```

Then:

```bash
hashcat -m <mode> -a 3 hash.txt '?d?d?d?d?d'
```

Then:

```bash
hashcat -m <mode> -a 3 hash.txt '?d?d?d?d?d?d'
```

Record the growth in candidate space.

---

# 43. Attack Mode Reference

```text
-a 0    Straight / dictionary
-a 1    Combination
-a 3    Brute-force / mask
-a 6    Hybrid wordlist + mask
-a 7    Hybrid mask + wordlist
```

Check your installed version:

```bash
hashcat --help
```

---

# 44. Example Laboratory Progression

For one synthetic target:

### Stage 1 — Dictionary

```bash
hashcat -m <mode> target.hash wordlist.txt
```

### Stage 2 — Rules

```bash
hashcat \
  -m <mode> \
  target.hash \
  wordlist.txt \
  -r rules/best64.rule
```

### Stage 3 — Targeted Mask

```bash
hashcat \
  -m <mode> \
  -a 3 \
  target.hash \
  'LAB?d?d?d?d'
```

### Stage 4 — Hybrid

```bash
hashcat \
  -m <mode> \
  -a 6 \
  target.hash \
  wordlist.txt \
  '?d?d'
```

### Stage 5 — Benchmark

```bash
hashcat -b -m <mode>
```

---

# 45. Automation

Basic laboratory script:

```bash
#!/bin/bash

MODE="$1"
TARGET="$2"
WORDLIST="$3"

hashcat \
    -m "$MODE" \
    -a 0 \
    "$TARGET" \
    "$WORDLIST"

hashcat \
    -m "$MODE" \
    --show \
    "$TARGET"
```

Save:

```bash
nano run-hashcat.sh
```

Make executable:

```bash
chmod +x run-hashcat.sh
```

Run:

```bash
./run-hashcat.sh <mode> target.hash wordlist.txt
```

---

# 46. Benchmark Script

```bash
#!/bin/bash

MODE="$1"

echo "[+] Hashcat version"
hashcat --version

echo
echo "[+] Devices"
hashcat -I

echo
echo "[+] Benchmark"
hashcat -b -m "$MODE"
```

Run:

```bash
chmod +x benchmark.sh
./benchmark.sh <mode>
```

Save the output for comparison between systems.

---

# 47. Troubleshooting

### "Token length exception"

Usually indicates that the input does not match the selected mode's expected format.

Check:

```bash
cat target.hash
```

Then:

```bash
hashcat --example-hashes
```

Verify the exact input structure.

---

### "Hash mode is not supported"

Check:

```bash
hashcat --help
```

Search:

```bash
hashcat --help | grep -i <keyword>
```

Your installed version/build may not support the required format.

---

### "Exhausted"

This means the selected candidate space was processed without recovering the target.

Possible reasons:

```text
Password not in wordlist
Wrong rules
Wrong mask
Incorrect attack model
Incorrect hash format
```

Do not interpret "Exhausted" as proof that the password is impossible to recover.

---

### Very Low Speed

Check:

```bash
hashcat -I
```

Then:

```bash
hashcat -b -m <mode>
```

Also inspect:

```bash
lspci | grep -Ei 'vga|3d|display'
```

Check system resources:

```bash
free -h
```

and:

```bash
top
```

---

# 48. Reproducible Experiment

Record:

```text
Hashcat Version:
Hash Mode:
Attack Mode:
Target:
Wordlist:
Rule:
Mask:
Device:
Driver:
Runtime:
Speed:
Candidates:
Result:
```

Example:

```text
Hashcat Version: [document]
Hash Mode:       [document]
Attack Mode:     0
Wordlist:        lab.txt
Rule:            best64.rule
Device:          [document]
Runtime:         [document]
Speed:           [document]
Result:          Recovered / Exhausted
```

---

# 49. Extreme Research Workflow

For a serious synthetic password-recovery experiment:

```text
Create Synthetic Target
        ↓
Identify Hash Format
        ↓
Verify Hash Mode
        ↓
Build Small Candidate Set
        ↓
Dictionary Baseline
        ↓
Dictionary + Rules
        ↓
Targeted Mask
        ↓
Hybrid Attack
        ↓
Benchmark
        ↓
Change Hardware
        ↓
Repeat
        ↓
Compare Results
```

Keep every variable documented.

---

# 50. Practical Comparison: John vs Hashcat

Use the same synthetic target and candidate set.

### John

```bash
john \
  --format=<format> \
  --wordlist=wordlist.txt \
  target.hash
```

### Hashcat

```bash
hashcat \
  -m <mode> \
  target.hash \
  wordlist.txt
```

Compare:

```text
Recovery
Runtime
Candidate Rate
CPU Usage
GPU Usage
Memory Usage
```

Do not compare results when the tools are using different attack configurations.

---

# 51. Final Command Checklist

```bash
# Installation
sudo apt install hashcat

# Version
hashcat --version

# Devices
hashcat -I

# Benchmark
hashcat -b

# Modes
hashcat --help

# Example formats
hashcat --example-hashes

# Dictionary
hashcat -m <mode> target.hash wordlist.txt

# Rules
hashcat -m <mode> target.hash wordlist.txt -r rules/best64.rule

# Mask
hashcat -m <mode> -a 3 target.hash '?d?d?d?d'

# Hybrid: word + mask
hashcat -m <mode> -a 6 target.hash wordlist.txt '?d?d'

# Hybrid: mask + word
hashcat -m <mode> -a 7 target.hash '?d?d' wordlist.txt

# Combination
hashcat -m <mode> -a 1 target.hash first.txt second.txt

# Show
hashcat -m <mode> target.hash --show

# Session
hashcat --session=lab01 -m <mode> target.hash wordlist.txt

# Restore
hashcat --session=lab01 --restore

# Benchmark specific mode
hashcat -b -m <mode>
```

---

# Safety

Use these commands only against:

* Your own password hashes
* Synthetic laboratory hashes
* Self-created encrypted data
* CTF/lab environments
* Explicitly authorized security assessments

Do not use recovered credentials from unauthorized systems or publish real credentials in research repositories.
