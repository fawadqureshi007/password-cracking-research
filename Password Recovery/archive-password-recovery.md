# Archive Password Recovery

> Practical laboratory guide for password-recovery testing against ZIP, 7z, and RAR archives that you own or are explicitly authorized to assess.

---

## Scope

This guide covers:

```text
ZIP
7z
RAR
```

The workflow is:

```text
Archive
   ↓
Identify Format
   ↓
Extract Password-Verification Data
   ↓
Identify Supported Format
   ↓
Choose Attack Method
   ↓
Run Offline Recovery
   ↓
Verify Password
   ↓
Document Results
```

All examples are intended for controlled laboratory files.

---

# 1. Install Tools

### Kali Linux

```bash
sudo apt update
sudo apt install john hashcat p7zip-full unrar
```

Check:

```bash
john --version
hashcat --version
7z
unrar
```

---

# 2. Create a Laboratory Archive

Create a test directory:

```bash
mkdir archive-lab
cd archive-lab
```

Create test data:

```bash
echo "Password recovery research lab" > research.txt
echo "Controlled test data" > notes.txt
```

Verify:

```bash
ls -la
```

---

# 3. Create a ZIP Test Archive

Create a password-protected ZIP:

```bash
zip -e lab.zip research.txt notes.txt
```

Enter a dedicated laboratory password when prompted.

Check:

```bash
file lab.zip
```

Test extraction:

```bash
unzip lab.zip
```

The archive should request the password.

---

# 4. Create a 7z Test Archive

Create:

```bash
7z a -p lab.7z research.txt notes.txt
```

For stronger encryption in a laboratory:

```bash
7z a -p -mhe=on lab.7z research.txt notes.txt
```

Check:

```bash
7z l lab.7z
```

Test extraction:

```bash
7z x lab.7z
```

---

# 5. Create a RAR Test Archive

If your environment provides RAR creation support:

```bash
rar a -p lab.rar research.txt notes.txt
```

Check:

```bash
file lab.rar
```

List:

```bash
unrar l lab.rar
```

Test extraction:

```bash
unrar x lab.rar
```

---

# 6. Identify the Archive

Before attempting recovery:

```bash
file lab.zip
file lab.7z
file lab.rar
```

Also inspect:

```bash
ls -lh lab.*
```

Do not assume an extension tells you the actual format.

---

# 7. ZIP Structure

Inspect a ZIP:

```bash
unzip -l lab.zip
```

You can also inspect the file signature:

```bash
xxd -l 16 lab.zip
```

A ZIP normally begins with a ZIP signature such as:

```text
50 4b
```

The exact structure depends on the archive.

---

# 8. 7z Structure

Inspect:

```bash
7z l lab.7z
```

Check:

```bash
xxd -l 16 lab.7z
```

The archive format contains its own encryption and metadata structure.

Do not treat 7z encryption as equivalent to ZIP encryption.

---

# 9. RAR Structure

Inspect:

```bash
unrar l lab.rar
```

Check:

```bash
xxd -l 16 lab.rar
```

RAR versions have different encryption implementations, so identify the format before choosing the recovery method.

---

# 10. John the Ripper

John can process several archive password formats through its supported formats and extraction utilities.

Check:

```bash
john --list=formats | grep -Ei 'zip|rar|7z'
```

Locate archive helper tools:

```bash
find /usr/share -iname '*2john*' 2>/dev/null
```

Common tools include:

```text
zip2john
rar2john
7z2john.pl
```

The exact location depends on your John installation.

---

# 11. ZIP → John

Locate:

```bash
find /usr/share -iname 'zip2john*' 2>/dev/null
```

Then extract the recovery material:

```bash
zip2john lab.zip > zip.hash
```

If `zip2john` is not in `$PATH`, use the path returned by `find`.

Check:

```bash
cat zip.hash
```

---

# 12. RAR → John

Locate:

```bash
find /usr/share -iname 'rar2john*' 2>/dev/null
```

Extract:

```bash
rar2john lab.rar > rar.hash
```

Check:

```bash
cat rar.hash
```

---

# 13. 7z → John

Locate:

```bash
find /usr/share -iname '7z2john*' 2>/dev/null
```

A common installation uses:

```bash
/usr/share/john/7z2john.pl
```

Extract:

```bash
perl /usr/share/john/7z2john.pl lab.7z > 7z.hash
```

Check:

```bash
cat 7z.hash
```

If the helper is located elsewhere, use that path instead.

---

# 14. Verify Extraction

Check the generated files:

```bash
ls -lh *.hash
```

Check size:

```bash
wc -c *.hash
```

A zero-byte file means extraction did not produce usable recovery material.

---

# 15. ZIP Dictionary Attack

Create a small laboratory wordlist:

```bash
cat > passwords.txt <<'EOF'
test
password
password123
archive
archive123
lab
lab123
research
research123
LabPass123
EOF
```

Run John:

```bash
john \
  --wordlist=passwords.txt \
  zip.hash
```

Show the result:

```bash
john --show zip.hash
```

---

# 16. RAR Dictionary Attack

```bash
john \
  --wordlist=passwords.txt \
  rar.hash
```

Show:

```bash
john --show rar.hash
```

---

# 17. 7z Dictionary Attack

```bash
john \
  --wordlist=passwords.txt \
  7z.hash
```

Show:

```bash
john --show 7z.hash
```

---

# 18. John Rules

Rules modify dictionary candidates.

Run:

```bash
john \
  --wordlist=passwords.txt \
  --rules \
  zip.hash
```

Check:

```bash
john --show zip.hash
```

The same approach can be used with supported RAR and 7z hashes.

---

# 19. Check John Formats

Before troubleshooting:

```bash
john --list=formats | grep -Ei 'zip|rar|7z'
```

This tells you which relevant formats your installed John build supports.

---

# 20. John Incremental Mode

John can generate candidates rather than using a predefined list.

Check:

```bash
john --list=incremental
```

Laboratory test:

```bash
john \
  --incremental \
  zip.hash
```

Stop:

```text
Ctrl+C
```

Check status:

```bash
john --status
```

Use small controlled password spaces when benchmarking.

---

# 21. Mask-Style Research

For predictable laboratory passwords, define the expected structure.

Example:

```text
LAB + four digits
```

Create the candidates:

```bash
for i in $(seq -w 0000 9999); do
    echo "LAB$i"
done > lab-mask.txt
```

Check:

```bash
head lab-mask.txt
tail lab-mask.txt
```

Then test:

```bash
john \
  --wordlist=lab-mask.txt \
  zip.hash
```

This is useful for measuring how predictable password structures affect recovery.

---

# 22. Numeric Candidate Space

Create a four-digit candidate list:

```bash
seq -w 0000 9999 > numeric.txt
```

Count:

```bash
wc -l numeric.txt
```

Run:

```bash
john \
  --wordlist=numeric.txt \
  zip.hash
```

This is a controlled way to demonstrate exhaustive search over a small keyspace.

---

# 23. Hashcat

Hashcat supports several archive formats.

First inspect the installed version:

```bash
hashcat --version
```

Search available modes:

```bash
hashcat --help | grep -Ei 'zip|rar|7z'
```

Also:

```bash
hashcat --example-hashes | grep -Ei 'zip|rar|7z'
```

Hashcat mode numbers can change or differ between formats and versions, so use the mode reported by your installed version.

---

# 24. Hashcat ZIP Test

After identifying the correct mode:

```bash
hashcat \
  -m <ZIP_MODE> \
  zip.hash \
  passwords.txt
```

Replace:

```text
<ZIP_MODE>
```

with the appropriate mode for your ZIP encryption format.

Show recovered credentials:

```bash
hashcat \
  -m <ZIP_MODE> \
  zip.hash \
  --show
```

---

# 25. Hashcat 7z Test

Find the supported mode:

```bash
hashcat --example-hashes | grep -i 7z
```

Then:

```bash
hashcat \
  -m <7Z_MODE> \
  7z.hash \
  passwords.txt
```

Show:

```bash
hashcat \
  -m <7Z_MODE> \
  7z.hash \
  --show
```

---

# 26. Hashcat RAR Test

Identify the exact RAR mode:

```bash
hashcat --example-hashes | grep -i rar
```

Then:

```bash
hashcat \
  -m <RAR_MODE> \
  rar.hash \
  passwords.txt
```

Show:

```bash
hashcat \
  -m <RAR_MODE> \
  rar.hash \
  --show
```

RAR versions and encryption schemes matter, so identify the actual format before selecting the mode.

---

# 27. Hashcat Benchmark

Benchmark your hardware:

```bash
hashcat -b
```

For a specific archive mode:

```bash
hashcat \
  -m <MODE> \
  -b
```

Record:

```text
Hashcat version
GPU
CPU
Mode
Speed
```

---

# 28. Hashcat Mask Attack

For a laboratory password consisting of four digits:

```bash
hashcat \
  -m <MODE> \
  zip.hash \
  -a 3 \
  '?d?d?d?d'
```

For:

```text
LAB + four digits
```

use:

```bash
hashcat \
  -m <MODE> \
  zip.hash \
  -a 3 \
  'LAB?d?d?d?d'
```

---

# 29. Hashcat Custom Character Set

Example laboratory search:

```bash
hashcat \
  -m <MODE> \
  zip.hash \
  -a 3 \
  -1 ?l?d \
  '?1?1?1?1?1?1'
```

This tests six-character passwords using lowercase letters and digits.

---

# 30. Hashcat Wordlist Attack

```bash
hashcat \
  -m <MODE> \
  zip.hash \
  passwords.txt
```

Monitor:

```text
s
```

Stop:

```text
q
```

---

# 31. Hashcat Rules

Use a rule file:

```bash
hashcat \
  -m <MODE> \
  zip.hash \
  passwords.txt \
  -r /usr/share/hashcat/rules/best64.rule
```

List installed rules:

```bash
ls /usr/share/hashcat/rules/
```

Rules can significantly increase the number of candidates generated from a small dictionary.

---

# 32. Hashcat Hybrid Attack

Word + two digits:

```bash
hashcat \
  -m <MODE> \
  zip.hash \
  -a 6 \
  passwords.txt \
  '?d?d'
```

Two digits + word:

```bash
hashcat \
  -m <MODE> \
  zip.hash \
  -a 7 \
  '?d?d' \
  passwords.txt
```

Use these only when the laboratory password pattern makes the search space reasonable.

---

# 33. Hashcat Sessions

Create a session:

```bash
hashcat \
  --session=archive-lab \
  -m <MODE> \
  zip.hash \
  passwords.txt
```

Check:

```bash
hashcat --session=archive-lab --status
```

Resume:

```bash
hashcat --session=archive-lab --restore
```

---

# 34. Archive Verification

After recovering a password, verify it against the original archive.

### ZIP

```bash
unzip -t lab.zip
```

Extract:

```bash
unzip lab.zip
```

### 7z

```bash
7z t lab.7z
```

Extract:

```bash
7z x lab.7z
```

### RAR

```bash
unrar t lab.rar
```

Extract:

```bash
unrar x lab.rar
```

Successful password recovery should be verified against the original archive.

---

# 35. ZIP Encryption Comparison

Create multiple laboratory ZIP files using different encryption configurations supported by your archive utility.

Record:

```text
Archive
Encryption
Password
Hash Format
Tool
Attack
Speed
Recovery
```

Do not assume all ZIP files have identical password-security properties.

---

# 36. 7z Encryption Experiment

Create:

```bash
7z a -p -mhe=on encrypted.7z research.txt
```

Then:

```bash
7z l encrypted.7z
```

Extract the recovery material using the appropriate John helper:

```bash
perl /usr/share/john/7z2john.pl encrypted.7z > encrypted-7z.hash
```

Run:

```bash
john \
  --wordlist=passwords.txt \
  encrypted-7z.hash
```

---

# 37. RAR Encryption Experiment

Create a laboratory archive with your installed RAR implementation.

Then inspect:

```bash
unrar l encrypted.rar
```

Extract:

```bash
rar2john encrypted.rar > encrypted-rar.hash
```

Run:

```bash
john \
  --wordlist=passwords.txt \
  encrypted-rar.hash
```

Verify:

```bash
john --show encrypted-rar.hash
```

---

# 38. Password Pattern Research

Create controlled passwords:

```text
lab123
Lab123
Lab1234
LabPass123
Research2026
LAB-2026
```

Create separate archives for each.

Measure:

```text
Length
Character classes
Pattern
Candidate space
Recovery method
Recovery time
```

This gives you comparable research data.

---

# 39. Wordlist Research

Create several controlled lists:

```text
small.txt
medium.txt
large.txt
```

Example:

```bash
wc -l small.txt
wc -l medium.txt
wc -l large.txt
```

Run the same archive against each.

Record:

```text
Wordlist size
Candidates tested
Speed
Runtime
Password found
```

---

# 40. Candidate-Space Research

For a password containing four digits:

```text
0000 → 9999
```

Total candidates:

```text
10,000
```

Generate:

```bash
seq -w 0000 9999 > 4digit.txt
```

For six digits:

```bash
seq -w 000000 999999 > 6digit.txt
```

Be aware that larger candidate spaces rapidly increase runtime.

---

# 41. Benchmark John

Run:

```bash
john --test
```

For archive-specific research, document:

```text
John version
CPU
Hash format
Attack mode
Candidate count
Speed
Runtime
```

---

# 42. Benchmark Hashcat

Run:

```bash
hashcat -b
```

For the relevant archive mode:

```bash
hashcat -m <MODE> -b
```

Record the same hardware and software information used for the John test.

---

# 43. John vs Hashcat

Use the same laboratory archive and candidate set.

Example:

```text
Archive:
lab.zip

Candidates:
10,000

John:
wordlist

Hashcat:
wordlist
```

Record:

```text
Tool
Version
Mode
Hardware
Candidates
Speed
Runtime
Result
```

Do not compare results if the tools were testing different candidate spaces.

---

# 44. Research Directory

Recommended structure:

```text
archive-lab/
├── archives/
│   ├── lab.zip
│   ├── lab.7z
│   └── lab.rar
│
├── hashes/
│   ├── zip.hash
│   ├── 7z.hash
│   └── rar.hash
│
├── wordlists/
│   ├── small.txt
│   ├── medium.txt
│   └── numeric.txt
│
└── results/
    ├── john.log
    └── hashcat.log
```

Keep recovered passwords and sensitive archive contents out of Git.

---

# 45. Logging

John:

```bash
john \
  --wordlist=passwords.txt \
  zip.hash \
  2>&1 | tee results/john-zip.log
```

Hashcat:

```bash
hashcat \
  -m <MODE> \
  zip.hash \
  passwords.txt \
  2>&1 | tee results/hashcat-zip.log
```

Record the command, tool version, hardware, and result.

---

# 46. Automation

Create:

```bash
nano run-archive-test.sh
```

Example:

```bash
#!/bin/bash

ARCHIVE="$1"
HASH="$2"
WORDLIST="$3"

echo "[+] Archive:  $ARCHIVE"
echo "[+] Hash:     $HASH"
echo "[+] Wordlist: $WORDLIST"

john \
    --wordlist="$WORDLIST" \
    "$HASH"

echo
echo "[+] Recovery result:"
john --show "$HASH"
```

Make executable:

```bash
chmod +x run-archive-test.sh
```

Run:

```bash
./run-archive-test.sh lab.zip zip.hash passwords.txt
```

---

# 47. Troubleshooting

## `zip2john` Not Found

```bash
find /usr/share -iname 'zip2john*' 2>/dev/null
```

Use the discovered executable.

---

## `rar2john` Not Found

```bash
find /usr/share -iname 'rar2john*' 2>/dev/null
```

---

## `7z2john.pl` Not Found

```bash
find /usr/share -iname '7z2john*' 2>/dev/null
```

---

## Hash File Is Empty

```bash
wc -c archive.hash
```

If zero:

```bash
file archive
```

Then verify that the archive format is supported by the extraction utility.

---

## John Says Format Is Unknown

Check:

```bash
john --list=formats | grep -Ei 'zip|rar|7z'
```

Verify the generated hash.

---

## Hashcat Says Token Length / Parsing Error

Check:

```bash
hashcat --example-hashes | grep -Ei 'zip|rar|7z'
```

Make sure:

```text
Correct hash
Correct mode
Correct archive format
Correct extraction method
```

are being used together.

---

## Password Not Found

Possible causes:

```text
Wrong wordlist
Wrong attack mode
Wrong archive format
Wrong extraction method
Password outside the tested keyspace
```

Validate the workflow using a laboratory password that you deliberately know.

---

# 48. Complete Practical Workflow

```text
Create Test Files
        ↓
Create Protected Archive
        ↓
Identify ZIP / 7z / RAR
        ↓
Test Manual Extraction
        ↓
Extract Recovery Material
        ↓
Verify Hash
        ↓
Identify John / Hashcat Format
        ↓
Run Small Wordlist
        ↓
Verify Recovery
        ↓
Run Rules
        ↓
Run Mask Attack
        ↓
Run Hybrid Attack
        ↓
Benchmark
        ↓
Compare Tools
        ↓
Document Results
```

---

# 49. Quick Reference

### Identify

```bash
file archive.zip
file archive.7z
file archive.rar
```

### ZIP

```bash
zip2john archive.zip > zip.hash
john --wordlist=passwords.txt zip.hash
john --show zip.hash
```

### RAR

```bash
rar2john archive.rar > rar.hash
john --wordlist=passwords.txt rar.hash
john --show rar.hash
```

### 7z

```bash
perl /usr/share/john/7z2john.pl archive.7z > 7z.hash
john --wordlist=passwords.txt 7z.hash
john --show 7z.hash
```

### John Formats

```bash
john --list=formats | grep -Ei 'zip|rar|7z'
```

### Hashcat Formats

```bash
hashcat --example-hashes | grep -Ei 'zip|rar|7z'
```

### Hashcat Wordlist

```bash
hashcat -m <MODE> archive.hash passwords.txt
```

### Hashcat Mask

```bash
hashcat -m <MODE> archive.hash -a 3 '?d?d?d?d'
```

### Hashcat Rules

```bash
hashcat \
  -m <MODE> \
  archive.hash \
  passwords.txt \
  -r /usr/share/hashcat/rules/best64.rule
```

### Verify ZIP

```bash
unzip -t archive.zip
```

### Verify 7z

```bash
7z t archive.7z
```

### Verify RAR

```bash
unrar t archive.rar
```

---

# 50. Research Checklist

```text
[ ] Archive belongs to the laboratory
[ ] Format identified
[ ] Encryption/protection type identified
[ ] Correct extraction utility selected
[ ] Recovery material extracted
[ ] Hash format verified
[ ] Small wordlist tested
[ ] Password pattern documented
[ ] Rules tested where appropriate
[ ] Mask tested where appropriate
[ ] Hashcat mode verified
[ ] Recovery verified against original archive
[ ] Runtime recorded
[ ] Hardware recorded
[ ] Tool versions recorded
[ ] Sensitive archives excluded from Git
[ ] Recovered passwords excluded from Git
```

---

## Scope

Use this research only for:

* Your own archives
* Authorized password-recovery work
* CTFs
* Controlled security laboratories
* Password-security research

Do not use recovered credentials or password-cracking techniques against archives belonging to other people or organizations without explicit authorization.
