# Office Password Recovery

> Practical laboratory guide for password recovery testing against Microsoft Office documents you own or are explicitly authorized to assess.

---

## 1. Scope

This guide covers password-protected:

```text
DOC
DOCX
XLS
XLSX
PPT
PPTX
```

The workflow is:

```text
Office File
    ↓
Identify Format
    ↓
Extract Password-Verification Data
    ↓
Identify Hashcat / John Format
    ↓
Choose Attack Strategy
    ↓
Run Offline Recovery
    ↓
Verify Recovered Password
```

No live authentication service is involved.

---

# 2. Install Required Tools

### Kali Linux

```bash
sudo apt update
sudo apt install john hashcat
```

Check:

```bash
john --version
hashcat --version
```

---

# 3. Create a Laboratory File

Create a test Office document.

For example:

```text
lab-document.docx
```

Set a password specifically for the experiment.

Example laboratory password:

```text
LabPass123
```

Do not use a real personal or organizational document.

---

# 4. Identify the File

```bash
file lab-document.docx
```

Check:

```bash
ls -lh lab-document.docx
```

For modern Office formats:

```bash
unzip -l lab-document.docx | head
```

A normal DOCX is a ZIP-based Office Open XML package.

---

# 5. DOCX Structure

A normal DOCX contains files such as:

```text
[Content_Types].xml
word/
_rels/
docProps/
```

Inspect:

```bash
unzip -l lab-document.docx
```

However, a document protected with **Office file encryption** is different from an ordinary ZIP archive.

Do not treat every DOCX password prompt as the same protection mechanism.

---

# 6. Identify the Protection Type

Office documents can use different forms of protection.

Examples:

```text
File encryption
Document protection
Worksheet protection
Workbook protection
VBA project protection
```

These are not equivalent.

For password-recovery research, first determine **what is actually protected**.

---

# 7. Modern Office Encryption

Modern Office files such as:

```text
.docx
.xlsx
.pptx
```

can use Microsoft's encryption mechanisms.

The file may begin with an OLE Compound File structure rather than the normal ZIP structure.

Check:

```bash
file encrypted.docx
```

If appropriate:

```bash
xxd -l 32 encrypted.docx
```

Do not rename the extension and assume the underlying format changed.

---

# 8. John the Ripper

John can process several Office password formats when the correct format is supported by the installed build.

First check:

```bash
john --list=formats | grep -i office
```

Also:

```bash
john --list=formats | grep -Ei 'office|ooxml|oldoffice'
```

---

# 9. Extract Office Hash Material

John distributions commonly include helper utilities for extracting password-recovery material from supported Office documents.

Locate them:

```bash
locate office2john
```

If `locate` is unavailable:

```bash
find /usr/share -iname '*office2john*' 2>/dev/null
```

Common Kali installations may place John utilities under:

```text
/usr/share/john/
```

Check:

```bash
ls /usr/share/john/
```

---

# 10. DOCX Extraction

For a supported encrypted DOCX:

```bash
python3 /usr/share/john/office2john.py encrypted.docx > office.hash
```

Check:

```bash
cat office.hash
```

If your installation places `office2john.py` elsewhere, use the path discovered earlier.

---

# 11. XLSX Extraction

```bash
python3 /usr/share/john/office2john.py encrypted.xlsx > office.hash
```

Check:

```bash
head -c 300 office.hash
echo
```

---

# 12. PPTX Extraction

```bash
python3 /usr/share/john/office2john.py encrypted.pptx > office.hash
```

Check:

```bash
head -c 300 office.hash
echo
```

---

# 13. DOC Extraction

Older Office formats use different underlying structures.

Check:

```bash
file encrypted.doc
```

Then try:

```bash
python3 /usr/share/john/office2john.py encrypted.doc > office.hash
```

If the installed utility does not recognize the format, check the current John documentation and available helper scripts.

---

# 14. XLS Extraction

```bash
python3 /usr/share/john/office2john.py encrypted.xls > office.hash
```

Check:

```bash
cat office.hash
```

---

# 15. PPT Extraction

```bash
python3 /usr/share/john/office2john.py encrypted.ppt > office.hash
```

---

# 16. Verify the Extracted Data

Before starting recovery:

```bash
wc -c office.hash
```

Then:

```bash
head -c 500 office.hash
echo
```

Make sure the file is not empty.

If extraction produced nothing useful, stop and determine the actual Office protection type.

---

# 17. Dictionary Attack with John

Create a small laboratory wordlist:

```bash
cat > office-passwords.txt <<'EOF'
test
password
office
office123
Lab123
LabPass123
research
research123
EOF
```

Run:

```bash
john \
  --wordlist=office-passwords.txt \
  office.hash
```

---

# 18. Show Recovered Password

```bash
john --show office.hash
```

Example result:

```text
encrypted.docx:LabPass123
```

The exact output depends on the Office format and John version.

---

# 19. Default John Wordlist

If your John installation provides a default wordlist:

```bash
john office.hash
```

Then check:

```bash
john --show office.hash
```

For controlled experiments, a small known test list is preferable because the experiment becomes repeatable.

---

# 20. Rules

John can transform dictionary candidates using rules.

Check available rules:

```bash
grep '^\[' /etc/john/john.conf
```

or:

```bash
grep '^\[' /usr/share/john/john.conf
```

Run with a selected rule section:

```bash
john \
  --wordlist=office-passwords.txt \
  --rules \
  office.hash
```

Then:

```bash
john --show office.hash
```

---

# 21. Incremental Testing

John supports incremental candidate generation.

Check available modes:

```bash
john --list=incremental
```

A laboratory experiment can use:

```bash
john \
  --incremental \
  office.hash
```

Stop it with:

```text
Ctrl+C
```

Then inspect:

```bash
john --status
```

This can become expensive very quickly as the candidate space grows.

---

# 22. Mask-Style Research with Hashcat

First identify the correct Hashcat mode.

```bash
hashcat --help | grep -i office
```

Search:

```bash
hashcat --example-hashes | grep -i office
```

Hashcat modes vary by Office format and version, so **do not hard-code a mode from an old guide**.

Use the mode reported by your installed version.

---

# 23. Hashcat Wordlist Attack

Once the correct mode is identified:

```bash
hashcat \
  -m <OFFICE_MODE> \
  office.hash \
  office-passwords.txt
```

Replace:

```text
<OFFICE_MODE>
```

with the mode supported by your installed Hashcat version.

---

# 24. Show Hashcat Result

```bash
hashcat \
  -m <OFFICE_MODE> \
  office.hash \
  --show
```

If a password was recovered, Hashcat displays the recovered credential material according to the hash format.

---

# 25. Benchmark the Office Mode

Before running a large experiment:

```bash
hashcat \
  -m <OFFICE_MODE> \
  -b
```

This gives you an idea of the available recovery performance for that mode and hardware.

---

# 26. Candidate Mask Testing

For a laboratory password following:

```text
LAB + 4 digits
```

a mask-based experiment can be used:

```bash
hashcat \
  -m <OFFICE_MODE> \
  office.hash \
  -a 3 \
  'LAB?d?d?d?d'
```

`?d` represents a digit in Hashcat's mask syntax.

This is useful when the test password structure is known.

---

# 27. Numeric Password Experiment

For a password consisting of four digits:

```bash
hashcat \
  -m <OFFICE_MODE> \
  office.hash \
  -a 3 \
  '?d?d?d?d'
```

This is practical for measuring a deliberately small search space.

---

# 28. Custom Character Set

Hashcat supports custom character sets.

Example:

```bash
hashcat \
  -m <OFFICE_MODE> \
  office.hash \
  -a 3 \
  -1 ?l?d \
  '?1?1?1?1?1?1'
```

This creates a controlled lowercase-letter/digit candidate space.

Use small spaces when testing performance.

---

# 29. Wordlist + Mask

A hybrid experiment can combine a wordlist with additional characters.

Example:

```bash
hashcat \
  -m <OFFICE_MODE> \
  office.hash \
  -a 6 \
  office-passwords.txt \
  '?d?d'
```

This tests:

```text
word + two digits
```

For example:

```text
Office12
Research42
Lab1234
```

---

# 30. Mask + Wordlist

The reverse hybrid mode can also be tested:

```bash
hashcat \
  -m <OFFICE_MODE> \
  office.hash \
  -a 7 \
  '?d?d' \
  office-passwords.txt
```

This tests:

```text
two digits + word
```

---

# 31. Rules with Hashcat

Hashcat supports rule-based candidate transformation.

Example:

```bash
hashcat \
  -m <OFFICE_MODE> \
  office.hash \
  office-passwords.txt \
  -r /usr/share/hashcat/rules/best64.rule
```

Check available rules:

```bash
ls /usr/share/hashcat/rules/
```

Common rule files may include:

```text
best64.rule
rockyou-30000.rule
```

Availability depends on your installation.

---

# 32. Hashcat Status

During a running job:

```text
s
```

Hashcat displays status information such as:

```text
Speed
Progress
Candidates
Time
Estimated completion
Recovered
```

Stop safely with:

```text
q
```

---

# 33. Hashcat Session

Create a named session:

```bash
hashcat \
  --session office-lab \
  -m <OFFICE_MODE> \
  office.hash \
  office-passwords.txt
```

Check status:

```bash
hashcat --session=office-lab --status
```

Resume:

```bash
hashcat --session=office-lab --restore
```

---

# 34. Potfile

Hashcat stores recovered results in its potfile.

Check the result:

```bash
hashcat \
  -m <OFFICE_MODE> \
  office.hash \
  --show
```

Keep the potfile private because it may contain recovered passwords.

---

# 35. John Session

Start:

```bash
john \
  --wordlist=office-passwords.txt \
  office.hash
```

Check status:

```bash
john --status
```

Show results:

```bash
john --show office.hash
```

John can also resume interrupted sessions using its session mechanism.

Check:

```bash
john --help
```

---

# 36. Verify the Password

Recovery is not complete until the recovered password is verified against the original laboratory document.

Open:

```text
encrypted.docx
```

Enter the recovered password.

Verify that:

```text
Document opens
Content is readable
Password is correct
```

For automated research, keep the original encrypted file unchanged.

---

# 37. Compare John and Hashcat

Use the same:

```text
Office file
Hash/extracted material
Password list
Hardware
Candidate space
```

Record:

```text
Tool
Version
Attack Mode
Candidates
Speed
Time
Recovered
```

Example:

```text
Tool: John
Mode: Wordlist
Candidates: 10,000
Time: ...
Result: ...

Tool: Hashcat
Mode: Wordlist
Candidates: 10,000
Time: ...
Result: ...
```

---

# 38. Password Pattern Experiment

Create several laboratory documents.

Use controlled passwords such as:

```text
Lab123
Lab1234
LabPass123
Research2026
LAB1234
```

Extract each independently.

Then test the same candidate sets.

Compare:

```text
Password
Length
Character Set
Pattern
Candidate Count
Recovery Time
```

---

# 39. Long Password Experiment

Create a separate document with a longer laboratory password.

Example:

```text
Correct-Horse-Lab-2026
```

Do not use this exact password for real security.

Use it to demonstrate how increasing length changes the candidate space.

---

# 40. Entropy Experiment

Create controlled passwords with increasing complexity:

```text
lab123
Lab123
Lab1234
LabPass123
Lab-Pass-2026
```

Record:

```text
Length
Character Classes
Pattern
Candidate Space
Recovery Result
```

The goal is to demonstrate why predictable structure can matter even when a password appears complex.

---

# 41. Legacy vs Modern Office

Create two laboratory documents:

```text
legacy.doc
modern.docx
```

Extract their password-verification data separately.

Compare:

```text
Format
Protection Scheme
Hash Representation
Supported Tools
Recovery Performance
```

Do not assume that `.doc` and `.docx` use the same password-protection mechanism.

---

# 42. XLSX Laboratory Test

Create:

```text
lab-workbook.xlsx
```

Protect it using file encryption.

Extract:

```bash
python3 /usr/share/john/office2john.py lab-workbook.xlsx > xlsx.hash
```

Inspect:

```bash
head -c 300 xlsx.hash
echo
```

Then use John or the corresponding Hashcat mode supported by your installed version.

---

# 43. PPTX Laboratory Test

Create:

```text
lab-presentation.pptx
```

Extract:

```bash
python3 /usr/share/john/office2john.py lab-presentation.pptx > pptx.hash
```

Then:

```bash
john --wordlist=office-passwords.txt pptx.hash
```

Check:

```bash
john --show pptx.hash
```

---

# 44. DOCX Laboratory Test

Create:

```text
lab-document.docx
```

Extract:

```bash
python3 /usr/share/john/office2john.py lab-document.docx > docx.hash
```

Run:

```bash
john \
  --wordlist=office-passwords.txt \
  docx.hash
```

Check:

```bash
john --show docx.hash
```

---

# 45. Batch Processing

Create:

```text
office-lab/
├── doc/
├── xls/
├── xlsx/
├── ppt/
├── pptx/
├── hashes/
├── wordlists/
└── results/
```

Example:

```bash
mkdir -p office-lab/{doc,xls,xlsx,ppt,pptx,hashes,wordlists,results}
```

Keep original documents separated from extracted hash material.

---

# 46. Batch Extraction

For supported Office files:

```bash
for file in office-lab/*/*; do
    [ -f "$file" ] || continue

    name=$(basename "$file")

    python3 /usr/share/john/office2john.py \
        "$file" \
        > "office-lab/hashes/${name}.hash"
done
```

Review:

```bash
ls -lh office-lab/hashes/
```

Only use this against your own laboratory files.

---

# 47. Research Logging

Create:

```bash
mkdir -p office-lab/results
```

Example:

```bash
john \
  --wordlist=office-lab/wordlists/passwords.txt \
  office-lab/hashes/lab-document.docx.hash \
  2>&1 | tee office-lab/results/john-docx.log
```

Then:

```bash
john --show \
  office-lab/hashes/lab-document.docx.hash
```

---

# 48. Benchmark Table

Use a consistent table:

```text
| File | Format | Tool | Attack | Candidates | Time | Result |
|------|--------|------|--------|------------|------|--------|
| A    | DOCX   | John | Wordlist | 10K | ... | ... |
| A    | DOCX   | Hashcat | Mask | ... | ... | ... |
| B    | XLSX   | John | Rules | ... | ... | ... |
| C    | PPTX   | Hashcat | Hybrid | ... | ... | ... |
```

Do not publish recovered passwords from real documents.

---

# 49. Troubleshooting

## `office2john.py` Not Found

Search:

```bash
find /usr/share -iname 'office2john.py' 2>/dev/null
```

Also:

```bash
find /opt -iname 'office2john.py' 2>/dev/null
```

Use the discovered path.

---

## Empty Hash File

Check:

```bash
wc -c office.hash
```

Then:

```bash
python3 /usr/share/john/office2john.py lab-document.docx
```

If no useful output is produced, determine whether the file actually uses Office file encryption or another protection mechanism.

---

## John Does Not Recognize the Hash

Check:

```bash
john --list=formats | grep -Ei 'office|ooxml|oldoffice'
```

Also verify that the extracted file was produced correctly.

---

## Hashcat Does Not Recognize the Hash

Check:

```bash
hashcat --example-hashes | grep -i office
```

Use the mode corresponding to the actual Office format.

Do not blindly use a mode copied from an outdated tutorial.

---

## Password Not Found

Possible reasons:

```text
Candidate list does not contain the password
Wrong attack mode
Wrong extraction method
Wrong protection type
Password pattern is larger than the tested space
```

Start with a known laboratory password and a tiny wordlist to validate the entire workflow.

---

# 50. Complete Laboratory Workflow

```text
Create Office File
        ↓
Set Known Laboratory Password
        ↓
Identify Office Format
        ↓
Determine Protection Type
        ↓
Extract Recovery Material
        ↓
Verify Extraction
        ↓
Select John / Hashcat
        ↓
Test Small Wordlist
        ↓
Verify Recovery
        ↓
Test Rules
        ↓
Test Masks
        ↓
Test Hybrid Attacks
        ↓
Benchmark
        ↓
Compare Results
        ↓
Document Findings
```

---

# 51. Quick Reference

### Identify File

```bash
file document.docx
```

### Locate Office Extraction Tool

```bash
find /usr/share -iname 'office2john.py' 2>/dev/null
```

### Extract

```bash
python3 /usr/share/john/office2john.py document.docx > office.hash
```

### John Wordlist

```bash
john --wordlist=passwords.txt office.hash
```

### John Rules

```bash
john --wordlist=passwords.txt --rules office.hash
```

### John Result

```bash
john --show office.hash
```

### List John Office Formats

```bash
john --list=formats | grep -Ei 'office|ooxml|oldoffice'
```

### Find Hashcat Office Modes

```bash
hashcat --example-hashes | grep -i office
```

### Hashcat Wordlist

```bash
hashcat -m <OFFICE_MODE> office.hash passwords.txt
```

### Hashcat Result

```bash
hashcat -m <OFFICE_MODE> office.hash --show
```

### Hashcat Mask

```bash
hashcat -m <OFFICE_MODE> office.hash -a 3 'LAB?d?d?d?d'
```

### Hashcat Rules

```bash
hashcat \
  -m <OFFICE_MODE> \
  office.hash \
  passwords.txt \
  -r /usr/share/hashcat/rules/best64.rule
```

---

# 52. Final Checklist

```text
[ ] Office file belongs to the lab
[ ] Protection type identified
[ ] Correct extraction method selected
[ ] Hash/extraction verified
[ ] John/Hashcat format verified
[ ] Small wordlist tested first
[ ] Attack parameters documented
[ ] Recovered password verified
[ ] Benchmark recorded
[ ] Sensitive files excluded from Git
[ ] Recovered passwords excluded from Git
```

---

## Scope

This guide is intended for:

* Password recovery from your own documents
* Authorized security assessments
* CTFs
* Isolated password-security laboratories
* Research into Office password protection

Do not use recovered-password techniques against documents you do not own or have permission to assess.
