# Dictionary Attacks

Dictionary attacks test a predefined list of password candidates against an offline hash or encrypted credential.

They are one of the first techniques to test when there is a reasonable chance that the password is based on common words, names, phrases, or predictable variations.

---

## 1. Attack Model

```text
Password
   ↓
Hash / Encrypted Credential
   ↓
Candidate from Wordlist
   ↓
Hash / Verification
   ↓
Compare
   ↓
Match / No Match
```

Unlike online guessing, the candidate testing happens locally against the authorized material.

---

## 2. Lab Setup

Install the required tools:

```bash
sudo apt update
sudo apt install john hashcat
```

Verify:

```bash
john --version
hashcat --version
```

---

## 3. Create a Test Hash

Create a disposable password:

```text
LabPassword2026!
```

Generate a SHA-256 hash:

```bash
printf '%s' 'LabPassword2026!' | sha256sum
```

Example output:

```text
<sha256-hash>  -
```

For the experiment, place the generated hash into:

```bash
echo '<sha256-hash>' > lab.hash
```

Do not use a real password.

---

## 4. Create a Small Wordlist

```bash
cat > passwords.txt <<'EOF'
password
password123
admin
admin123
welcome
qwerty
LabPassword
LabPassword2026
LabPassword2026!
EOF
```

Check the number of candidates:

```bash
wc -l passwords.txt
```

Display the list:

```bash
cat passwords.txt
```

---

## 5. John the Ripper

Run a dictionary attack:

```bash
john \
    --format=Raw-SHA256 \
    --wordlist=passwords.txt \
    lab.hash
```

Show the result:

```bash
john --show lab.hash
```

If the laboratory password is present in the wordlist, John should recover it.

---

## 6. John With a Larger Wordlist

The same workflow can be used with a larger authorized wordlist:

```bash
john \
    --format=Raw-SHA256 \
    --wordlist=/path/to/wordlist.txt \
    lab.hash
```

Check progress:

```bash
john --status
```

Show recovered passwords:

```bash
john --show lab.hash
```

---

## 7. John Rules

A plain dictionary only tests the words exactly as they appear.

Rules generate variations from those words.

Example:

```bash
john \
    --format=Raw-SHA256 \
    --wordlist=passwords.txt \
    --rules \
    lab.hash
```

Common transformations can produce candidates such as:

```text
password
Password
PASSWORD
password1
Password1
password!
Password2026
```

The exact transformations depend on the selected John rule configuration.

---

## 8. John Rule Configuration

List John configuration information:

```bash
john --list=rules
```

Check the configuration:

```bash
john --config
```

Available rule names depend on the installed John version and configuration.

Use the rules available on your own system rather than assuming a particular rule set exists.

---

## 9. John Incremental Testing

Dictionary attacks should generally be attempted before a large exhaustive search when a realistic candidate list exists.

For research:

```bash
john \
    --format=Raw-SHA256 \
    --incremental \
    lab.hash
```

Monitor:

```bash
john --status
```

Stop:

```text
Ctrl+C
```

---

## 10. John Sessions

Create a named session:

```bash
john \
    --session=dictionary-lab \
    --format=Raw-SHA256 \
    --wordlist=passwords.txt \
    lab.hash
```

Check:

```bash
john --status=dictionary-lab
```

Restore:

```bash
john --restore=dictionary-lab
```

---

## 11. Hashcat

Hashcat uses attack mode `0` for a straight dictionary attack.

For raw SHA-256:

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    passwords.txt
```

Check recovered results:

```bash
hashcat \
    -m 1400 \
    lab.hash \
    --show
```

Confirm the mode on your installed version:

```bash
hashcat --example-hashes | grep -A 3 -i 'SHA-256'
```

Do not blindly use a mode number when working with another hash format.

---

## 12. Hashcat With a Larger Wordlist

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    /path/to/wordlist.txt
```

Basic status:

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    passwords.txt \
    --status
```

---

## 13. Hashcat Rules

Dictionary candidates can be transformed using rules.

Example:

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    passwords.txt \
    -r /usr/share/hashcat/rules/best64.rule
```

Show the result:

```bash
hashcat \
    -m 1400 \
    lab.hash \
    --show
```

The available rule files depend on the Hashcat installation.

Check:

```bash
ls /usr/share/hashcat/rules/
```

---

## 14. Generate Candidate Variations

Instead of immediately using a huge wordlist, build a focused laboratory list.

Example:

```bash
cat > base.txt <<'EOF'
password
security
redteam
research
Lab
EOF
```

Then use the appropriate rules to generate variations.

This allows experiments to measure:

```text
Base candidates
Generated candidates
Recovery rate
Runtime
```

---

## 15. Wordlist Quality

A wordlist is not simply judged by its size.

Compare:

```text
10,000 highly relevant candidates
```

against:

```text
10,000,000 unrelated candidates
```

The smaller list may be more effective if it better represents the password-generation pattern being researched.

Measure:

```text
Candidate count
Unique candidates
Relevant candidates
Generation method
Recovery rate
Runtime
```

---

## 16. Duplicate Candidates

Check duplicate lines:

```bash
sort passwords.txt | uniq -d
```

Create a deduplicated list:

```bash
sort -u passwords.txt > passwords-clean.txt
```

Compare:

```bash
wc -l passwords.txt
wc -l passwords-clean.txt
```

Removing duplicates prevents wasting candidate tests.

---

## 17. Inspect Wordlist Size

```bash
wc -l passwords-clean.txt
```

File size:

```bash
du -h passwords-clean.txt
```

First candidates:

```bash
head passwords-clean.txt
```

Last candidates:

```bash
tail passwords-clean.txt
```

---

## 18. Sort a Wordlist

Alphabetical sorting:

```bash
sort passwords.txt > passwords-sorted.txt
```

Numeric sorting is useful when candidate files contain numbers:

```bash
sort -n passwords.txt
```

For large research datasets, keep the original wordlist unchanged and generate a separate processed copy.

---

## 19. Candidate Generation

A research workflow can generate candidates from controlled inputs.

Example:

```bash
cat > names.txt <<'EOF'
alice
bob
charlie
EOF
```

And:

```bash
cat > years.txt <<'EOF'
2024
2025
2026
EOF
```

Generate combinations:

```bash
while read name; do
    while read year; do
        printf '%s%s\n' "$name" "$year"
    done < years.txt
done < names.txt > generated.txt
```

Inspect:

```bash
cat generated.txt
```

This creates a controlled candidate set without targeting real accounts.

---

## 20. Prefix and Suffix Research

Create a laboratory wordlist:

```bash
cat > words.txt <<'EOF'
security
redteam
research
cyber
EOF
```

Generate a suffix:

```bash
while read word; do
    printf '%s2026\n' "$word"
done < words.txt
```

Generate a prefix:

```bash
while read word; do
    printf 'Cyber%s\n' "$word"
done < words.txt
```

Redirect results when creating a real candidate file:

```bash
while read word; do
    printf '%s2026\n' "$word"
done < words.txt > candidates.txt
```

---

## 21. Combining Multiple Sources

For research, candidate sources can include:

```text
Generic dictionary
Synthetic project terms
Synthetic usernames
Synthetic organization terms
Synthetic dates
Synthetic keywords
```

Keep each source separate:

```text
wordlists/
├── base.txt
├── names.txt
├── years.txt
├── keywords.txt
└── generated.txt
```

This makes the experiment reproducible.

---

## 22. Dictionary + Known Suffix

Suppose the controlled experiment assumes:

```text
word + 2026
```

Generate:

```bash
while read word; do
    printf '%s2026\n' "$word"
done < base.txt > candidates.txt
```

Run:

```bash
john \
    --format=Raw-SHA256 \
    --wordlist=candidates.txt \
    lab.hash
```

Or:

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    candidates.txt
```

---

## 23. Dictionary + Rules vs Plain Dictionary

Run the plain dictionary first:

```bash
john \
    --format=Raw-SHA256 \
    --wordlist=base.txt \
    lab.hash
```

Then test rules:

```bash
john \
    --format=Raw-SHA256 \
    --wordlist=base.txt \
    --rules \
    lab.hash
```

Record:

```text
Attack
Candidate source
Candidate count
Runtime
Result
```

This provides a measurable comparison.

---

## 24. Measure Candidate Coverage

For every experiment, record:

```text
Wordlist:
Candidate count:
Unique candidates:
Transformations:
Known password pattern:
Password recovered:
Position:
Runtime:
```

If the correct password occurs at candidate position `N`, that provides useful information about the ordering of the dictionary.

---

## 25. Candidate Ordering

The same wordlist can produce different results depending on candidate ordering.

Compare:

```text
Common-first
Alphabetical
Frequency-based
Context-specific
Generated variations
```

For a controlled experiment:

```bash
wc -l candidates.txt
```

Then record where the known laboratory password appears.

---

## 26. Benchmarking

John:

```bash
john --test
```

Hashcat:

```bash
hashcat -b
```

Record:

```text
CPU
GPU
RAM
Operating system
Tool version
Hash format
Measured speed
```

Do not compare benchmark results from different hardware as if they were equivalent.

---

## 27. Estimate Search Cost

If a dictionary contains:

```text
1,000,000 candidates
```

and the measured testing speed is:

```text
100,000 candidates/second
```

then the theoretical full-list time is:

```text
1,000,000 / 100,000 = 10 seconds
```

Real experiments may differ because of:

```text
Candidate generation
Rules
I/O
Hardware utilization
KDF cost
Thermal throttling
Tool overhead
```

Use measured runtime in final research results.

---

## 28. Dictionary Attack vs Brute Force

Dictionary:

```text
password
welcome
security
redteam
...
```

Brute force:

```text
aaaa
aaab
aaac
...
```

Dictionary attacks can cover likely passwords much earlier when the password is based on recognizable words.

Brute force becomes useful when there is little information about the password structure and the search space is manageable.

---

## 29. Dictionary Attack vs Mask Attack

Dictionary:

```text
security
redteam
research
```

Mask:

```text
Security2026
Security2027
Security2028
...
```

Use a mask when the structure is known.

Use a dictionary when the likely password itself can be represented by candidate words.

---

## 30. Dictionary Attack vs Rule Attack

Plain dictionary:

```text
password
security
redteam
```

Rules:

```text
Password
Password1
Password!
Security2026
Redteam2026
```

Rules increase coverage without requiring every variation to be manually stored.

---

## 31. Large Wordlists

Do not automatically assume:

```text
larger wordlist = better result
```

A large irrelevant wordlist can consume considerable time before reaching useful candidates.

For research, compare:

```text
Small focused list
Large generic list
Focused + rules
Large + rules
```

Record actual results.

---

## 32. Synthetic Context Lists

For authorized research, create context-specific words from synthetic data.

Example:

```text
project:
RedLab

keywords:
security
research
testing

year:
2026
```

Create:

```text
RedLab
RedLab2026
RedLabSecurity
RedLabSecurity2026
RedLabResearch
```

Do not collect or use private information from real people to construct unauthorized password candidates.

---

## 33. Unicode and Encoding

Password candidates may contain non-ASCII characters.

Check encoding:

```bash
file passwords.txt
```

Inspect bytes:

```bash
xxd passwords.txt | head
```

Keep the encoding consistent between:

```text
Candidate generator
Wordlist
Cracking tool
Target representation
```

Encoding mistakes can make a correct candidate fail verification.

---

## 34. Line Endings

Check:

```bash
file passwords.txt
```

Convert Windows line endings when necessary:

```bash
dos2unix passwords.txt
```

Verify:

```bash
file passwords.txt
```

Unexpected carriage returns can create incorrect candidates.

---

## 35. Split a Large Wordlist

For large laboratory datasets:

```bash
split -l 1000000 passwords.txt chunk-
```

Check:

```bash
ls -lh chunk-*
```

This allows experiments to be divided into manageable batches.

---

## 36. Preserve the Original Dataset

Use:

```text
wordlists/
├── original/
│   └── source.txt
├── processed/
│   └── cleaned.txt
└── generated/
    └── candidates.txt
```

Never overwrite the original research data.

---

## 37. Reproducible Experiment

Record:

```text
Experiment ID:
Date:
Hash format:
Tool:
Tool version:
Hardware:
Wordlist:
Wordlist size:
Rules:
Attack mode:
Candidate count:
Start time:
End time:
Result:
```

Example:

```text
Experiment ID: DICT-001
Tool: John
Attack: Dictionary
Wordlist: lab-small.txt
Candidates: 1000
Rules: No
Result: Recovered
```

Replace example values with actual measurements.

---

## 38. Research Matrix

Run controlled comparisons:

| Experiment | Wordlist | Rules | Candidates | Result | Runtime |
| ---------- | -------- | ----: | ---------: | ------ | ------- |
| DICT-01    | Small    |    No |     Record | Record | Record  |
| DICT-02    | Small    |   Yes |     Record | Record | Record  |
| DICT-03    | Large    |    No |     Record | Record | Record  |
| DICT-04    | Large    |   Yes |     Record | Record | Record  |

Use measured values from your own laboratory.

---

## 39. Troubleshooting

### No Password Found

Check:

```text
Correct hash format
Correct hash
Correct wordlist
Candidate encoding
Candidate ordering
Tool configuration
Attack mode
```

---

### Unexpectedly Slow

Check:

```bash
john --test
```

or:

```bash
hashcat -b
```

Then investigate:

```text
CPU/GPU utilization
KDF cost
Thermal throttling
Rule complexity
Candidate generation
```

---

### Duplicate Candidates

Run:

```bash
sort -u passwords.txt > passwords-clean.txt
```

Then:

```bash
wc -l passwords.txt
wc -l passwords-clean.txt
```

---

## 40. Practical Workflow

```text
Create Authorized Laboratory Hash
             ↓
Identify Hash Format
             ↓
Create Small Focused Wordlist
             ↓
Remove Duplicates
             ↓
Run Plain Dictionary
             ↓
Record Result
             ↓
Run Rules
             ↓
Record Result
             ↓
Expand Candidate Sources
             ↓
Benchmark
             ↓
Compare Results
             ↓
Document Findings
```

---

## 41. Quick Reference

### Create SHA-256 Test Hash

```bash
printf '%s' 'LabPassword2026!' | sha256sum
```

### Create Wordlist

```bash
cat > passwords.txt <<'EOF'
password
password123
LabPassword2026!
EOF
```

### Count Candidates

```bash
wc -l passwords.txt
```

### Remove Duplicates

```bash
sort -u passwords.txt > passwords-clean.txt
```

### John Dictionary

```bash
john \
    --format=Raw-SHA256 \
    --wordlist=passwords-clean.txt \
    lab.hash
```

### John Rules

```bash
john \
    --format=Raw-SHA256 \
    --wordlist=passwords-clean.txt \
    --rules \
    lab.hash
```

### John Results

```bash
john --show lab.hash
```

### Hashcat Dictionary

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    passwords-clean.txt
```

### Hashcat Results

```bash
hashcat \
    -m 1400 \
    lab.hash \
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

## 42. Research Checklist

```text
[ ] Authorized laboratory material used
[ ] Synthetic password created
[ ] Hash format identified
[ ] Wordlist created
[ ] Wordlist size recorded
[ ] Duplicate candidates removed
[ ] Plain dictionary tested
[ ] Rule-based dictionary tested
[ ] Candidate generation documented
[ ] Encoding checked
[ ] Benchmark recorded
[ ] Runtime recorded
[ ] Result verified
[ ] Original research data preserved
[ ] No real credentials included
```

---

## 43. Key Findings to Document

A useful dictionary-attack study should answer:

```text
Which wordlist was used?
How many candidates did it contain?
How many unique candidates remained?
Was the password recovered?
How many candidates were tested?
How long did recovery take?
What hardware was used?
Did rules improve recovery?
Did a focused list outperform a larger generic list?
```

The goal is to produce **measurable, reproducible password-security research**, not simply a successful recovery.
