# John the Ripper — Practical Guide

> Practical John the Ripper reference for authorized labs, CTFs, and self-created password-protected files.

---

## 1. Install

### Kali / Debian

```bash
sudo apt update
sudo apt install john
```

Check:

```bash
john --version
john --help
```

Build information:

```bash
john --list=build-info
```

---

## 2. See Supported Formats

```bash
john --list=formats
```

Search for a format:

```bash
john --list=formats | grep -i ntlm
john --list=formats | grep -i bcrypt
john --list=formats | grep -i pdf
john --list=formats | grep -i zip
```

---

## 3. Create a Lab Hash

Never start with someone else's credential database.

Create a synthetic password and generate a corresponding test hash using the appropriate laboratory tool or application.

Then save it:

```bash
nano hash.txt
```

Example:

```text
<YOUR-LAB-HASH>
```

---

## 4. Test Format Detection

Try John against the test file:

```bash
john hash.txt
```

If the format is known, specify it:

```bash
john --format=<format> hash.txt
```

Example:

```bash
john --format=nt hash.txt
```

Use the format name reported by your installed build.

---

## 5. Dictionary Attack

Basic:

```bash
john --wordlist=wordlist.txt hash.txt
```

Common system wordlist:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

If `rockyou.txt` is compressed:

```bash
sudo gzip -dk /usr/share/wordlists/rockyou.txt.gz
```

Then:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

---

## 6. Specify the Format

```bash
john --format=<format> \
     --wordlist=wordlist.txt \
     hash.txt
```

Example:

```bash
john --format=nt \
     --wordlist=wordlist.txt \
     hash.txt
```

---

## 7. Show Recovered Passwords

```bash
john --show hash.txt
```

Show results for a specific format:

```bash
john --format=<format> --show hash.txt
```

---

## 8. Check Current Progress

While John is running, press:

```text
q
```

or:

```text
Ctrl+C
```

For a named session:

```bash
john --status=lab01
```

---

## 9. Named Sessions

Start:

```bash
john \
  --session=lab01 \
  --wordlist=wordlist.txt \
  hash.txt
```

Check:

```bash
john --status=lab01
```

Resume:

```bash
john --restore=lab01
```

Stop a long-running terminal session safely with:

```text
Ctrl+C
```

Then restore it later.

---

# 10. Rules

Dictionary only:

```bash
john --wordlist=wordlist.txt hash.txt
```

Dictionary + default rules:

```bash
john \
  --wordlist=wordlist.txt \
  --rules \
  hash.txt
```

Specify a rule section:

```bash
john \
  --wordlist=wordlist.txt \
  --rules=<rule-section> \
  hash.txt
```

Inspect the configuration to see available rule sections:

```bash
john --list=rules
```

---

# 11. Custom Rule Research

John's configuration contains rule definitions.

Locate the configuration:

```bash
john --list=build-info
```

Inspect the installed configuration file.

Back it up before modifying:

```bash
cp john.conf john.conf.bak
```

Create a dedicated laboratory rule section and test it only against synthetic targets.

Example concept:

```text
Base word
   ↓
Case modification
   ↓
Number modification
   ↓
Symbol modification
   ↓
Candidate
```

Run the selected rule section:

```bash
john \
  --wordlist=lab.txt \
  --rules=<your-rule-section> \
  hash.txt
```

---

# 12. Single Mode

John can derive candidates from information contained in supported input formats.

```bash
john --single hash.txt
```

Specify a format when required:

```bash
john \
  --single \
  --format=<format> \
  hash.txt
```

---

# 13. Incremental Mode

Start an incremental search:

```bash
john --incremental hash.txt
```

Specify a configured incremental mode:

```bash
john \
  --incremental=<mode> \
  hash.txt
```

View available modes through the installed configuration.

Incremental searches can become extremely large, so use a small synthetic target first.

---

# 14. Incremental With Format

```bash
john \
  --format=<format> \
  --incremental=<mode> \
  hash.txt
```

---

# 15. Custom Wordlist

Create:

```bash
nano lab.txt
```

Example synthetic candidates:

```text
admin
password
research
security
lab
example
test
```

Run:

```bash
john --wordlist=lab.txt hash.txt
```

---

# 16. Sort and Remove Duplicates

Large candidate lists can contain duplicates.

```bash
sort wordlist.txt | uniq > clean-wordlist.txt
```

Count candidates:

```bash
wc -l clean-wordlist.txt
```

Run:

```bash
john --wordlist=clean-wordlist.txt hash.txt
```

---

# 17. Password Pattern Research

If you know the password structure of your **lab target**, reduce the candidate space instead of blindly searching everything.

Example laboratory structure:

```text
LAB + 4 digits
```

The candidate space is:

```text
LAB0000
LAB0001
LAB0002
...
LAB9999
```

Generate candidates:

```bash
for i in $(seq -w 0 9999); do
    echo "LAB$i"
done > lab-candidates.txt
```

Run:

```bash
john \
  --wordlist=lab-candidates.txt \
  hash.txt
```

This is useful for studying how known password structure affects recovery time.

---

# 18. Candidate Generation With Python

For more complex synthetic patterns:

```python
with open("lab-candidates.txt", "w") as f:
    for year in range(2020, 2031):
        for number in range(100):
            f.write(f"Research{year}{number:02d}!\n")
```

Then:

```bash
john \
  --wordlist=lab-candidates.txt \
  hash.txt
```

Record:

```bash
wc -l lab-candidates.txt
```

---

# 19. Mask-Style Research

For a known synthetic structure, generate the unknown portion yourself.

Example:

```text
LAB + 4 digits
```

Generate:

```bash
seq -w 0 9999 | sed 's/^/LAB/' > candidates.txt
```

Run:

```bash
john --wordlist=candidates.txt hash.txt
```

This approach makes the candidate space explicit and easy to benchmark.

---

# 20. Hybrid Candidate Generation

Create a synthetic base list:

```bash
cat > base.txt <<'EOF'
security
research
testing
password
EOF
```

Generate combinations:

```bash
for word in $(cat base.txt); do
    for year in 2024 2025 2026; do
        echo "${word}${year}!"
    done
done > hybrid.txt
```

Run:

```bash
john --wordlist=hybrid.txt hash.txt
```

---

# 21. Multiple Targets

Put multiple authorized laboratory hashes in one file:

```text
hash1
hash2
hash3
```

Run:

```bash
john --wordlist=wordlist.txt hashes.txt
```

View recovered results:

```bash
john --show hashes.txt
```

---

# 22. Pot File

Find the John installation/configuration information:

```bash
john --list=build-info
```

The pot file stores previously recovered results.

Never commit it to Git:

```bash
echo "*.pot" >> .gitignore
```

Also avoid committing files containing real recovered credentials.

---

# 23. Encrypted PDF

Use a **PDF you created yourself** and protect it with a known laboratory password.

First inspect available PDF-related formats:

```bash
john --list=formats | grep -i pdf
```

Depending on the John build, the required extraction utility may be available as:

```bash
pdf2john
```

or a similarly named executable.

Locate it:

```bash
command -v pdf2john
find /usr -type f -iname '*pdf2john*' 2>/dev/null
```

Extract the verification data:

```bash
pdf2john protected.pdf > pdf.hash
```

Then run:

```bash
john \
  --wordlist=wordlist.txt \
  pdf.hash
```

View:

```bash
john --show pdf.hash
```

The exact extraction utility can vary by John package/build.

---

# 24. ZIP

Create a ZIP archive yourself.

Check available ZIP-related formats:

```bash
john --list=formats | grep -i zip
```

Locate the extraction utility:

```bash
command -v zip2john
find /usr -type f -iname '*zip2john*' 2>/dev/null
```

Extract:

```bash
zip2john protected.zip > zip.hash
```

Run:

```bash
john \
  --wordlist=wordlist.txt \
  zip.hash
```

Show:

```bash
john --show zip.hash
```

---

# 25. RAR

Check support:

```bash
john --list=formats | grep -i rar
```

Locate:

```bash
command -v rar2john
find /usr -type f -iname '*rar2john*' 2>/dev/null
```

Extract:

```bash
rar2john protected.rar > rar.hash
```

Run:

```bash
john \
  --wordlist=wordlist.txt \
  rar.hash
```

Show:

```bash
john --show rar.hash
```

---

# 26. 7-Zip

Check:

```bash
john --list=formats | grep -i 7z
```

Locate:

```bash
command -v 7z2john
find /usr -type f -iname '*7z2john*' 2>/dev/null
```

Extract:

```bash
7z2john protected.7z > 7z.hash
```

Run:

```bash
john \
  --wordlist=wordlist.txt \
  7z.hash
```

Show:

```bash
john --show 7z.hash
```

---

# 27. Office Documents

For a laboratory Office document:

```bash
john --list=formats | grep -Ei 'office|doc|xlsx|ppt'
```

Locate the appropriate extraction utility if included in the installed John package.

Typical workflow:

```text
Office File
    ↓
Extraction Utility
    ↓
John Input
    ↓
Candidate Attack
    ↓
Recovery
```

Then:

```bash
john --wordlist=wordlist.txt extracted.hash
```

---

# 28. SSH Private-Key Passphrase

Use a private key generated specifically for your lab.

Check available formats:

```bash
john --list=formats | grep -i ssh
```

Locate the extractor:

```bash
find /usr -type f -iname '*ssh2john*' 2>/dev/null
```

Extract:

```bash
ssh2john protected_key > ssh.hash
```

Run:

```bash
john --wordlist=wordlist.txt ssh.hash
```

Show:

```bash
john --show ssh.hash
```

---

# 29. Keepass

Use a laboratory KeePass database.

Check:

```bash
john --list=formats | grep -i keepass
```

Locate the appropriate extraction utility:

```bash
find /usr -type f -iname '*keepass*john*' 2>/dev/null
```

Extract the verification data and run:

```bash
john \
  --wordlist=wordlist.txt \
  keepass.hash
```

---

# 30. Format-Specific Execution

When John detects multiple possibilities or you want reproducible results:

```bash
john \
  --format=<format> \
  --wordlist=wordlist.txt \
  target.hash
```

For example:

```bash
john \
  --format=<lab-format> \
  --wordlist=wordlist.txt \
  target.hash
```

Always use the format name returned by:

```bash
john --list=formats
```

---

# 31. Benchmark

Run John's built-in benchmark:

```bash
john --test
```

Benchmark a specific format where supported:

```bash
john --test --format=<format>
```

Record:

```text
John version
Format
CPU
GPU
RAM
Candidate rate
```

---

# 32. Performance Testing

Create a controlled target.

Run a baseline:

```bash
time john \
  --wordlist=wordlist.txt \
  target.hash
```

Record candidate count:

```bash
wc -l wordlist.txt
```

Run again with another configuration and compare:

```text
Candidate Count
Runtime
Candidate Rate
Recovery Result
```

Keep the target, hardware, and candidate set identical when comparing configurations.

---

# 33. Long-Running Research

Start a named session:

```bash
john \
  --session=experiment01 \
  --wordlist=wordlist.txt \
  target.hash
```

Check:

```bash
john --status=experiment01
```

Resume:

```bash
john --restore=experiment01
```

Show results:

```bash
john --show target.hash
```

---

# 34. Automation

A simple laboratory shell workflow:

```bash
#!/bin/bash

TARGET="$1"
WORDLIST="$2"
FORMAT="$3"

john \
    --format="$FORMAT" \
    --wordlist="$WORDLIST" \
    "$TARGET"

john \
    --format="$FORMAT" \
    --show \
    "$TARGET"
```

Save:

```bash
nano run-john.sh
```

Make executable:

```bash
chmod +x run-john.sh
```

Run against your laboratory target:

```bash
./run-john.sh target.hash wordlist.txt <format>
```

---

# 35. Troubleshooting

### "No password hashes loaded"

Check the format:

```bash
john --list=formats
```

Then explicitly specify it:

```bash
john --format=<format> target.hash
```

Check that the input file actually contains the extracted hash:

```bash
cat target.hash
```

---

### Format Not Available

Search:

```bash
john --list=formats | grep -i <keyword>
```

If it does not appear, your installed build may not support that format.

---

### Wordlist Attack Finishes Immediately

Check:

```bash
wc -l wordlist.txt
```

Check the target:

```bash
cat target.hash
```

Check whether the password is actually represented in your test candidates.

---

### Slow Performance

Check system load:

```bash
top
```

or:

```bash
htop
```

Check available memory:

```bash
free -h
```

Check CPU information:

```bash
lscpu
```

Check GPU information where applicable:

```bash
lspci | grep -Ei 'vga|3d|display'
```

---

# 36. Researching Attack Efficiency

For every experiment, record:

```text
Target Format:
Attack Mode:
Candidate Source:
Candidate Count:
Rules:
Hardware:
John Version:
Start Time:
End Time:
Result:
```

Then compare:

```text
Dictionary
        vs
Dictionary + Rules
        vs
Targeted Candidates
        vs
Incremental
```

The goal is to determine which strategy covers the target candidate space most efficiently.

---

# 37. Practical Attack Progression

Use this order when testing a synthetic target:

```text
1. Format Identification
        ↓
2. Small Targeted Wordlist
        ↓
3. Full Wordlist
        ↓
4. Wordlist + Rules
        ↓
5. Targeted Candidate Generation
        ↓
6. Incremental Search
        ↓
7. Advanced Candidate Research
        ↓
8. Performance Benchmark
```

Do not jump directly to exhaustive search when a much smaller candidate space is known.

---

# 38. Extreme Research Workflow

For a serious laboratory experiment:

```text
Create Target
      ↓
Identify Format
      ↓
Document Parameters
      ↓
Generate Candidate Models
      ↓
Run Baseline
      ↓
Measure Rate
      ↓
Optimize Candidate Generation
      ↓
Run Second Test
      ↓
Compare Results
      ↓
Repeat With Different Parameters
      ↓
Document Final Results
```

Example:

```text
Experiment:
Predictable 10-character synthetic password

Test A:
Generic dictionary

Test B:
Dictionary + rules

Test C:
Known pattern candidates

Test D:
Incremental

Record:
Candidates
Runtime
Candidate rate
Recovery result
```

This gives you an actual research comparison rather than just a successful crack.

---

# 39. Useful Commands — Quick Reference

```bash
john --version
john --help
john --list=formats
john --list=build-info
john --list=rules

john target.hash

john --format=<format> target.hash

john --wordlist=wordlist.txt target.hash

john --wordlist=wordlist.txt --rules target.hash

john --single target.hash

john --incremental target.hash

john --session=lab01 --wordlist=wordlist.txt target.hash

john --status=lab01

john --restore=lab01

john --show target.hash

john --test

john --test --format=<format>
```

---

# 40. Final Practical Workflow

For an authorized laboratory target:

```bash
# 1. Check John
john --version

# 2. Check formats
john --list=formats

# 3. Inspect target
cat target.hash

# 4. Test the format
john --format=<format> target.hash

# 5. Dictionary
john --format=<format> \
     --wordlist=wordlist.txt \
     target.hash

# 6. Rules
john --format=<format> \
     --wordlist=wordlist.txt \
     --rules \
     target.hash

# 7. Show result
john --format=<format> \
     --show \
     target.hash
```

For encrypted laboratory files:

```text
Protected File
      ↓
Correct *2john extractor
      ↓
Extracted Hash
      ↓
Format Identification
      ↓
Wordlist
      ↓
Rules
      ↓
Targeted Candidates
      ↓
Incremental / Advanced Search
      ↓
Result
      ↓
Benchmark
```

---

# Safety

Use these techniques only against:

* Your own password hashes
* Your own encrypted files
* CTF/lab targets
* Systems you have explicit authorization to test

Never use recovered credentials from unauthorized systems.

Do not publish real credentials, private keys, password databases, or sensitive pot-file contents.
