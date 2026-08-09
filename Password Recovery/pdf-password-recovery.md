# PDF Password Recovery

> Practical laboratory guide for password-recovery testing against PDF documents in controlled and authorized environments.

---

## Scope

This guide covers:

```text
PDF encryption
PDF password extraction
John the Ripper
Hashcat
Wordlist attacks
Mask attacks
Rule-based attacks
Hybrid attacks
Benchmarking
Password verification
```

Workflow:

```text
PDF
 ↓
Identify Protection
 ↓
Extract Recovery Data
 ↓
Identify Hash Format
 ↓
Choose Tool
 ↓
Run Controlled Attack
 ↓
Verify Password
 ↓
Record Results
```

---

# 1. Install Tools

### Kali Linux

```bash
sudo apt update
sudo apt install john hashcat poppler-utils qpdf
```

Check:

```bash
john --version
hashcat --version
qpdf --version
pdfinfo -v
```

---

# 2. Create a Laboratory PDF

Create or obtain a PDF specifically for the experiment:

```text
lab-document.pdf
```

Use a password that you deliberately created for the laboratory.

Example:

```text
LabPass123
```

Do not use a personal, financial, academic, or organizational PDF.

---

# 3. Identify the PDF

```bash
file lab-document.pdf
```

Check:

```bash
ls -lh lab-document.pdf
```

Use:

```bash
pdfinfo lab-document.pdf
```

Look for encryption information.

---

# 4. Inspect PDF Encryption

Run:

```bash
qpdf --show-encryption lab-document.pdf
```

This can reveal information about the PDF's encryption configuration.

Example information may include:

```text
R
P
User password
Owner password
Encryption
Permissions
```

The exact output depends on the PDF and qpdf version.

---

# 5. Check Whether the PDF Opens

Try:

```bash
qpdf --check lab-document.pdf
```

If the document requires a password, use the normal PDF viewer to confirm the password prompt.

The goal is to establish that the laboratory file is actually protected before starting recovery.

---

# 6. PDF Encryption Types

PDF files can use different encryption revisions and configurations.

Common families include:

```text
RC4-based PDF encryption
AES-based PDF encryption
Older PDF security revisions
Modern PDF encryption revisions
```

Do not select a John or Hashcat mode based only on the `.pdf` extension.

Identify the actual extracted format first.

---

# 7. Locate `pdf2john`

John the Ripper commonly provides a PDF extraction utility.

Find it:

```bash
find /usr/share -iname 'pdf2john*' 2>/dev/null
```

Also check:

```bash
find /opt -iname 'pdf2john*' 2>/dev/null
```

Common locations include:

```text
/usr/share/john/pdf2john.pl
```

Your installation may use a different path.

---

# 8. Extract PDF Recovery Data

For a supported PDF:

```bash
python3 /usr/share/john/pdf2john.pl lab-document.pdf > pdf.hash
```

If your installation provides the script as Perl:

```bash
perl /usr/share/john/pdf2john.pl lab-document.pdf > pdf.hash
```

Use whichever interpreter matches the installed script.

---

# 9. Check the Extracted Data

```bash
ls -lh pdf.hash
```

Check size:

```bash
wc -c pdf.hash
```

View:

```bash
head -c 500 pdf.hash
echo
```

The output should contain PDF password-verification material.

Do not publish the extracted hash from a real document.

---

# 10. Identify the John Format

Check available formats:

```bash
john --list=formats | grep -i pdf
```

You can also search:

```bash
john --list=formats | grep -Ei 'pdf|pdf24|pdf14'
```

The exact format names depend on your John build.

---

# 11. Create a Small Wordlist

Create a controlled test list:

```bash
cat > pdf-passwords.txt <<'EOF'
test
password
password123
pdf
pdf123
document
document123
lab
lab123
LabPass123
research
research123
EOF
```

Check:

```bash
cat pdf-passwords.txt
```

Count:

```bash
wc -l pdf-passwords.txt
```

---

# 12. John Wordlist Attack

Run:

```bash
john \
  --wordlist=pdf-passwords.txt \
  pdf.hash
```

Check status:

```bash
john --status
```

Show recovered passwords:

```bash
john --show pdf.hash
```

---

# 13. Verify the Password

After John reports a recovery, verify it against the original PDF.

Open:

```text
lab-document.pdf
```

Enter the recovered password.

The document should open successfully.

Do not consider the recovery complete until the password has been verified against the original file.

---

# 14. John Rules

Rules transform dictionary candidates.

Run:

```bash
john \
  --wordlist=pdf-passwords.txt \
  --rules \
  pdf.hash
```

Then:

```bash
john --show pdf.hash
```

Rules can generate many additional candidates from a relatively small wordlist.

---

# 15. Inspect Available Rules

Depending on your installation:

```bash
grep '^\[' /etc/john/john.conf
```

or:

```bash
grep '^\[' /usr/share/john/john.conf
```

You can also inspect John configuration:

```bash
john --list=sections
```

Use the rules available in your installed version.

---

# 16. Incremental Mode

John can generate candidates automatically.

Check:

```bash
john --list=incremental
```

Run a controlled test:

```bash
john \
  --incremental \
  pdf.hash
```

Stop:

```text
Ctrl+C
```

Check:

```bash
john --status
```

Incremental attacks can become extremely large, so use them carefully when benchmarking.

---

# 17. Numeric Password Experiment

For a deliberately created four-digit laboratory password:

```bash
seq -w 0000 9999 > four-digit.txt
```

Check:

```bash
wc -l four-digit.txt
```

Run:

```bash
john \
  --wordlist=four-digit.txt \
  pdf.hash
```

This gives a controlled 10,000-candidate experiment.

---

# 18. Pattern-Based Password Research

Suppose the laboratory password follows:

```text
PDF + four digits
```

Generate the candidates:

```bash
for i in $(seq -w 0000 9999); do
    echo "PDF$i"
done > pdf-pattern.txt
```

Check:

```bash
head pdf-pattern.txt
tail pdf-pattern.txt
```

Run:

```bash
john \
  --wordlist=pdf-pattern.txt \
  pdf.hash
```

This demonstrates why predictable password structures can dramatically reduce a search space.

---

# 19. Hashcat

Hashcat can process supported PDF password formats.

First inspect:

```bash
hashcat --version
```

Search available modes:

```bash
hashcat --help | grep -i pdf
```

Also:

```bash
hashcat --example-hashes | grep -i pdf
```

Use the mode corresponding to the actual PDF encryption format.

Do not blindly copy a PDF mode number from an old tutorial.

---

# 20. Hashcat Wordlist Attack

After identifying the correct mode:

```bash
hashcat \
  -m <PDF_MODE> \
  pdf.hash \
  pdf-passwords.txt
```

Replace:

```text
<PDF_MODE>
```

with the mode supported by your installed Hashcat version.

Show results:

```bash
hashcat \
  -m <PDF_MODE> \
  pdf.hash \
  --show
```

---

# 21. Hashcat Mask Attack

For a four-digit laboratory password:

```bash
hashcat \
  -m <PDF_MODE> \
  pdf.hash \
  -a 3 \
  '?d?d?d?d'
```

For:

```text
PDF + four digits
```

use:

```bash
hashcat \
  -m <PDF_MODE> \
  pdf.hash \
  -a 3 \
  'PDF?d?d?d?d'
```

---

# 22. Custom Character Set

Example:

```bash
hashcat \
  -m <PDF_MODE> \
  pdf.hash \
  -a 3 \
  -1 ?l?d \
  '?1?1?1?1?1?1'
```

This creates a controlled six-character search space using lowercase letters and digits.

---

# 23. Hashcat Rules

Use a rule file:

```bash
hashcat \
  -m <PDF_MODE> \
  pdf.hash \
  pdf-passwords.txt \
  -r /usr/share/hashcat/rules/best64.rule
```

Check installed rules:

```bash
ls /usr/share/hashcat/rules/
```

The exact rule files available depend on your installation.

---

# 24. Hashcat Hybrid Attack

Word + two digits:

```bash
hashcat \
  -m <PDF_MODE> \
  pdf.hash \
  -a 6 \
  pdf-passwords.txt \
  '?d?d'
```

Two digits + word:

```bash
hashcat \
  -m <PDF_MODE> \
  pdf.hash \
  -a 7 \
  '?d?d' \
  pdf-passwords.txt
```

Use hybrid attacks when your laboratory password pattern makes the search space reasonable.

---

# 25. Hashcat Status

During a running attack:

```text
s
```

Hashcat displays information such as:

```text
Speed
Progress
Candidates
Recovered
Time
Estimated completion
```

Stop:

```text
q
```

---

# 26. Hashcat Sessions

Create a session:

```bash
hashcat \
  --session=pdf-lab \
  -m <PDF_MODE> \
  pdf.hash \
  pdf-passwords.txt
```

Check:

```bash
hashcat --session=pdf-lab --status
```

Resume:

```bash
hashcat --session=pdf-lab --restore
```

---

# 27. Hashcat Benchmark

General benchmark:

```bash
hashcat -b
```

PDF-specific benchmark:

```bash
hashcat \
  -m <PDF_MODE> \
  -b
```

Record:

```text
Hashcat version
PDF mode
CPU
GPU
Speed
```

---

# 28. Compare John and Hashcat

Use the same:

```text
PDF
Extracted hash
Password list
Candidate space
Hardware
```

Example:

```text
PDF:
lab-document.pdf

Candidates:
10,000

John:
Wordlist

Hashcat:
Wordlist
```

Record:

```text
Tool
Version
Mode
Attack
Candidates
Speed
Runtime
Result
```

Do not compare results when the candidate spaces are different.

---

# 29. Password-Length Experiment

Create several laboratory PDFs using controlled passwords:

```text
lab1
Lab123
Lab1234
LabPass123
PDF-Lab-2026
```

Extract each independently.

Record:

```text
Password
Length
Character Classes
Pattern
Candidate Space
Recovery Time
```

The experiment demonstrates how password length and structure affect recovery.

---

# 30. Character-Set Experiment

Create controlled passwords using:

```text
Lowercase
Uppercase
Digits
Symbols
Mixed characters
```

Example:

```text
labpassword
LabPassword
LabPassword1
LabPassword1!
```

Run equivalent candidate-space experiments.

Record the difference in search requirements.

---

# 31. Wordlist Experiment

Create:

```text
small.txt
medium.txt
large.txt
```

Count:

```bash
wc -l small.txt
wc -l medium.txt
wc -l large.txt
```

Run the same PDF against each list.

Record:

```text
Wordlist
Candidate Count
Speed
Runtime
Recovery
```

---

# 32. PDF Encryption Comparison

Create multiple laboratory PDFs using different encryption settings supported by your PDF software.

Record:

```text
PDF Version
Encryption Revision
Cipher
Key Size
Password
John Format
Hashcat Mode
Recovery Result
```

Use `qpdf` to inspect supported encryption information:

```bash
qpdf --show-encryption lab-document.pdf
```

---

# 33. `qpdf` Inspection

Check:

```bash
qpdf --check lab-document.pdf
```

Show encryption:

```bash
qpdf --show-encryption lab-document.pdf
```

This is useful before extracting the recovery material because it confirms what kind of PDF protection you are testing.

---

# 34. Batch PDF Testing

Recommended laboratory structure:

```text
pdf-lab/
├── pdf/
│   ├── test-01.pdf
│   ├── test-02.pdf
│   └── test-03.pdf
│
├── hashes/
│   ├── test-01.hash
│   ├── test-02.hash
│   └── test-03.hash
│
├── wordlists/
│   ├── small.txt
│   ├── numeric.txt
│   └── patterns.txt
│
└── results/
    ├── john/
    └── hashcat/
```

Create it:

```bash
mkdir -p pdf-lab/{pdf,hashes,wordlists,results/john,results/hashcat}
```

---

# 35. Logging John

Run:

```bash
john \
  --wordlist=pdf-lab/wordlists/small.txt \
  pdf-lab/hashes/test-01.hash \
  2>&1 | tee pdf-lab/results/john/test-01.log
```

Show:

```bash
john --show pdf-lab/hashes/test-01.hash
```

---

# 36. Logging Hashcat

Run:

```bash
hashcat \
  -m <PDF_MODE> \
  pdf-lab/hashes/test-01.hash \
  pdf-lab/wordlists/small.txt \
  2>&1 | tee pdf-lab/results/hashcat/test-01.log
```

Show:

```bash
hashcat \
  -m <PDF_MODE> \
  pdf-lab/hashes/test-01.hash \
  --show
```

---

# 37. Automated PDF Test

Create:

```bash
nano run-pdf-test.sh
```

Example:

```bash
#!/bin/bash

PDF="$1"
HASH="$2"
WORDLIST="$3"

echo "[+] PDF:      $PDF"
echo "[+] Hash:     $HASH"
echo "[+] Wordlist: $WORDLIST"

john \
    --wordlist="$WORDLIST" \
    "$HASH"

echo
echo "[+] Result:"
john --show "$HASH"
```

Make executable:

```bash
chmod +x run-pdf-test.sh
```

Run:

```bash
./run-pdf-test.sh \
    lab-document.pdf \
    pdf.hash \
    pdf-passwords.txt
```

---

# 38. Recovery Verification

Once a password is recovered:

1. Open the original PDF.
2. Enter the recovered password.
3. Confirm the PDF opens.
4. Confirm the expected content is accessible.
5. Record the result.

Do not modify the original laboratory PDF during verification.

---

# 39. Troubleshooting

## `pdf2john` Not Found

Search:

```bash
find /usr/share -iname 'pdf2john*' 2>/dev/null
```

Also:

```bash
find /opt -iname 'pdf2john*' 2>/dev/null
```

Use the path returned by the search.

---

## Empty Hash

Check:

```bash
wc -c pdf.hash
```

Then:

```bash
file lab-document.pdf
```

Inspect:

```bash
qpdf --show-encryption lab-document.pdf
```

If extraction fails, determine whether the PDF uses a supported encryption/protection scheme.

---

## John Does Not Recognize the Hash

Check:

```bash
john --list=formats | grep -i pdf
```

Verify:

```bash
head -c 500 pdf.hash
echo
```

Make sure the extraction utility and John version are compatible.

---

## Hashcat Does Not Recognize the Hash

Search:

```bash
hashcat --example-hashes | grep -i pdf
```

Verify:

```text
Correct PDF format
Correct extraction
Correct Hashcat mode
Correct hash file
```

---

## Password Not Found

Possible causes:

```text
Wrong wordlist
Wrong mask
Wrong attack mode
Wrong PDF format
Wrong extraction method
Password outside tested keyspace
```

Validate your workflow with a PDF whose laboratory password you deliberately know.

---

# 40. Complete Practical Workflow

```text
Create Laboratory PDF
        ↓
Set Known Password
        ↓
Identify PDF
        ↓
Inspect Encryption
        ↓
Locate pdf2john
        ↓
Extract Recovery Material
        ↓
Verify Hash
        ↓
Identify John Format
        ↓
Identify Hashcat Mode
        ↓
Small Wordlist
        ↓
Verify Recovery
        ↓
Rules
        ↓
Mask
        ↓
Hybrid
        ↓
Benchmark
        ↓
Compare John / Hashcat
        ↓
Document Results
```

---

# 41. Quick Reference

### Identify PDF

```bash
file document.pdf
pdfinfo document.pdf
qpdf --check document.pdf
```

### Inspect Encryption

```bash
qpdf --show-encryption document.pdf
```

### Locate `pdf2john`

```bash
find /usr/share -iname 'pdf2john*' 2>/dev/null
```

### Extract

```bash
python3 /usr/share/john/pdf2john.pl document.pdf > pdf.hash
```

### John Formats

```bash
john --list=formats | grep -i pdf
```

### John Wordlist

```bash
john --wordlist=passwords.txt pdf.hash
```

### John Rules

```bash
john --wordlist=passwords.txt --rules pdf.hash
```

### John Result

```bash
john --show pdf.hash
```

### Hashcat PDF Modes

```bash
hashcat --example-hashes | grep -i pdf
```

### Hashcat Wordlist

```bash
hashcat -m <PDF_MODE> pdf.hash passwords.txt
```

### Hashcat Mask

```bash
hashcat -m <PDF_MODE> pdf.hash -a 3 '?d?d?d?d'
```

### Hashcat Rules

```bash
hashcat \
  -m <PDF_MODE> \
  pdf.hash \
  passwords.txt \
  -r /usr/share/hashcat/rules/best64.rule
```

### Hashcat Result

```bash
hashcat -m <PDF_MODE> pdf.hash --show
```

---

# 42. Research Checklist

```text
[ ] PDF belongs to the laboratory
[ ] PDF format identified
[ ] Encryption identified
[ ] qpdf inspection completed
[ ] pdf2john located
[ ] Recovery material extracted
[ ] Hash verified
[ ] John format verified
[ ] Hashcat mode verified
[ ] Small wordlist tested
[ ] Rules tested
[ ] Mask tested
[ ] Hybrid tested where appropriate
[ ] Password verified against original PDF
[ ] Runtime recorded
[ ] Hardware recorded
[ ] Tool versions recorded
[ ] Sensitive PDF excluded from Git
[ ] Extracted hash excluded from public Git
[ ] Recovered password excluded from Git
```

---

## Scope

Use this guide for:

* Your own PDF files
* Authorized password-recovery work
* CTFs
* Isolated security laboratories
* PDF password-security research

Do not use password-recovery techniques against PDFs belonging to other people or organizations without explicit authorization.
