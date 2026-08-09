# Hybrid Attacks

Practical guide to combining wordlists and masks for offline password-recovery research.

Hybrid attacks are useful when part of a password is predictable as a word and another part follows a structured pattern.

Examples:

```text
password123
redteam2026
admin01
2026security
```

Instead of searching the entire password space, hybrid attacks combine a known wordlist with a controlled mask.

All examples use disposable laboratory hashes.

---

## 1. Attack Model

There are two main Hashcat hybrid modes:

```text
-a 6
Wordlist + Mask

-a 7
Mask + Wordlist
```

### Wordlist + Mask

```text
Word
 ↓
Mask
 ↓
Candidate
```

Example:

```text
redteam + 2026
```

produces:

```text
redteam2026
```

### Mask + Wordlist

```text
Mask
 ↓
Word
 ↓
Candidate
```

Example:

```text
2026 + security
```

produces:

```text
2026security
```

---

# Lab Setup

## 2. Install Hashcat

```bash
sudo apt update
sudo apt install hashcat
```

Verify:

```bash
hashcat --version
```

Check attack modes:

```bash
hashcat --help
```

---

## 3. Create a Laboratory Wordlist

```bash
cat > words.txt <<'EOF'
password
admin
redteam
security
cyber
summer
dragon
EOF
```

Check:

```bash
cat words.txt
```

Count:

```bash
wc -l words.txt
```

---

## 4. Create a Laboratory Password

Use a disposable password.

Example:

```text
redteam2026
```

Generate a SHA-256 hash:

```bash
printf '%s' 'redteam2026' | sha256sum
```

Save it:

```bash
echo '<sha256-hash>' > lab.hash
```

---

# Wordlist + Mask

## 5. Basic Hybrid Attack

Hashcat attack mode `6` combines a wordlist with a mask.

Syntax:

```bash
hashcat \
    -m <hash-mode> \
    -a 6 \
    <hash-file> \
    <wordlist> \
    <mask>
```

Example:

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?d?d'
```

This tests candidates such as:

```text
password00
password01
password02
...
admin00
admin01
...
redteam00
redteam01
...
```

---

## 6. Word + One Digit

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?d'
```

For every word, Hashcat appends:

```text
0
1
2
3
...
9
```

---

## 7. Word + Two Digits

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?d?d'
```

For each word:

```text
word00
word01
word02
...
word99
```

---

## 8. Word + Three Digits

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?d?d?d'
```

Each word receives:

```text
000
001
002
...
999
```

---

## 9. Word + Four Digits

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?d?d?d?d'
```

This produces:

```text
word0000
word0001
...
word9999
```

---

## 10. Word + Lowercase Character

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?l'
```

Example:

```text
admina
adminb
adminc
...
```

---

## 11. Word + Uppercase Character

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?u'
```

---

## 12. Word + Mixed Pattern

Example:

```text
word + uppercase + digit + digit
```

Use:

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?u?d?d'
```

Candidates can look like:

```text
passwordA00
passwordA01
passwordB00
...
```

---

## 13. Word + Special Character

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?s'
```

This tests each word with available special characters.

---

# Mask + Wordlist

## 14. Basic Reverse Hybrid

Attack mode `7` places the mask before the wordlist candidate.

Syntax:

```bash
hashcat \
    -m <hash-mode> \
    -a 7 \
    <hash-file> \
    <mask> \
    <wordlist>
```

Example:

```bash
hashcat \
    -m 1400 \
    -a 7 \
    lab.hash \
    '?d?d' \
    words.txt
```

Candidates include:

```text
00password
01password
02password
...
00admin
01admin
...
```

---

## 15. Two Digits + Word

```bash
hashcat \
    -m 1400 \
    -a 7 \
    lab.hash \
    '?d?d' \
    words.txt
```

Useful for structures such as:

```text
01security
02security
99security
```

---

## 16. Four Digits + Word

```bash
hashcat \
    -m 1400 \
    -a 7 \
    lab.hash \
    '?d?d?d?d' \
    words.txt
```

Example candidates:

```text
2024security
2025security
2026security
```

---

## 17. Mixed Prefix

Example:

```text
uppercase + digit + word
```

Run:

```bash
hashcat \
    -m 1400 \
    -a 7 \
    lab.hash \
    '?u?d' \
    words.txt
```

---

# Custom Character Sets

## 18. Custom Suffix Alphabet

Define:

```bash
hashcat \
    -m 1400 \
    -a 6 \
    -1 '123456' \
    lab.hash \
    words.txt \
    '?1?1'
```

The mask generates two characters using:

```text
1
2
3
4
5
6
```

---

## 19. Custom Prefix Alphabet

For mask + word:

```bash
hashcat \
    -m 1400 \
    -a 7 \
    -1 '123456' \
    lab.hash \
    '?1?1' \
    words.txt
```

---

## 20. Multiple Custom Character Sets

Example:

```bash
hashcat \
    -m 1400 \
    -a 6 \
    -1 'ABC' \
    -2 '123' \
    lab.hash \
    words.txt \
    '?1?2'
```

The suffix contains:

```text
A1
A2
A3
B1
B2
B3
C1
C2
C3
```

---

# Keyspace

## 21. Basic Formula

For hybrid attacks:

```text
Total Candidates =
Wordlist Size × Mask Keyspace
```

Example:

```text
1,000 words
+
2 digits
```

Mask keyspace:

```text
10 × 10
= 100
```

Total:

```text
1,000 × 100
= 100,000 candidates
```

---

## 22. Three-Digit Example

```text
Wordlist:
5,000 words

Mask:
?d?d?d
```

Mask keyspace:

```text
10^3
= 1,000
```

Total candidates:

```text
5,000 × 1,000
= 5,000,000
```

---

## 23. Character-Set Example

Suppose:

```text
Wordlist = 2,000 words
Mask = ?l?d
```

Mask keyspace:

```text
26 × 10
= 260
```

Total:

```text
2,000 × 260
= 520,000 candidates
```

---

# Candidate Generation

## 24. Preview Candidates

Use `--stdout` before running a large recovery experiment.

Example:

```bash
hashcat \
    -a 6 \
    words.txt \
    '?d?d' \
    --stdout
```

Save:

```bash
hashcat \
    -a 6 \
    words.txt \
    '?d?d' \
    --stdout > hybrid-candidates.txt
```

Count:

```bash
wc -l hybrid-candidates.txt
```

---

## 25. Preview Reverse Hybrid

```bash
hashcat \
    -a 7 \
    '?d?d' \
    words.txt \
    --stdout
```

Save:

```bash
hashcat \
    -a 7 \
    '?d?d' \
    words.txt \
    --stdout > prefix-candidates.txt
```

---

## 26. Check Unique Candidates

```bash
sort -u hybrid-candidates.txt > unique.txt
```

Count:

```bash
wc -l hybrid-candidates.txt
wc -l unique.txt
```

This identifies duplicate candidates.

---

# Practical Experiments

## 27. Word + Year Experiment

Create:

```bash
cat > years.txt <<'EOF'
2024
2025
2026
EOF
```

A simple hybrid attack can instead use a four-digit mask:

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?d?d?d?d'
```

If the research only concerns specific years, a dedicated candidate list may be considerably smaller.

---

## 28. Word + Two Digits

Laboratory target:

```text
redteam42
```

Run:

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?d?d'
```

Show result:

```bash
hashcat \
    -m 1400 \
    lab.hash \
    --show
```

---

## 29. Prefix + Word Experiment

Laboratory target:

```text
2026security
```

Run:

```bash
hashcat \
    -m 1400 \
    -a 7 \
    lab.hash \
    '?d?d?d?d' \
    words.txt
```

Show:

```bash
hashcat \
    -m 1400 \
    lab.hash \
    --show
```

---

## 30. Word + Mixed Suffix

Target structure:

```text
word + uppercase + two digits
```

Command:

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?u?d?d'
```

---

## 31. Word + Special + Digit

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?s?d'
```

---

## 32. Digit + Word + Digit

A single hybrid attack does not directly place a mask on both sides of the dictionary word.

For research, generate the desired structure explicitly or use appropriate Hashcat attack features after checking your installed version:

```bash
hashcat --help
```

Do not assume every arbitrary pattern can be represented by `-a 6` or `-a 7`.

---

# Rules + Hybrid

## 33. Rule-Based Wordlist + Mask

Rules and hybrid attacks can be combined by preparing the dictionary candidates first.

Generate transformed candidates:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r /usr/share/hashcat/rules/best64.rule \
    --stdout > transformed.txt
```

Then use the transformed list:

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    transformed.txt \
    '?d?d'
```

This creates:

```text
Word transformations
        +
Mask suffix
        ↓
Expanded candidate space
```

Measure the size before launching the recovery.

---

## 34. Candidate Explosion

Suppose:

```text
words.txt = 10,000 words
```

Rules produce:

```text
50 candidates per word
```

The transformed list can approach:

```text
500,000 candidates
```

Adding:

```text
?d?d
```

creates another:

```text
× 100
```

Potential candidate count:

```text
50,000,000
```

Always estimate the workload before combining large wordlists, large rule sets, and masks.

---

# Benchmarking

## 35. Hashcat Benchmark

```bash
hashcat -b
```

Record:

```text
GPU
CPU
Hash algorithm
Speed
Driver
Hashcat version
```

---

## 36. Measure Candidate Generation

```bash
time hashcat \
    -a 6 \
    words.txt \
    '?d?d' \
    --stdout > /dev/null
```

Record:

```text
Wordlist size
Mask
Generated candidates
Generation time
```

---

## 37. Measure Recovery

Run:

```bash
time hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?d?d'
```

Record:

```text
Hash mode
Wordlist
Mask
Keyspace
Speed
Runtime
Result
```

---

# Sessions

## 38. Named Session

```bash
hashcat \
    --session=hybrid-lab \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?d?d'
```

Restore:

```bash
hashcat --session=hybrid-lab --restore
```

Or:

```bash
hashcat --restore
```

---

# Research Comparisons

## 39. Dictionary vs Hybrid

Test:

```text
A → dictionary only
B → dictionary + 1 digit
C → dictionary + 2 digits
D → dictionary + 4 digits
```

Record:

| Test | Attack          | Keyspace | Candidates | Runtime | Result |
| ---- | --------------- | -------: | ---------: | ------: | ------ |
| A    | Dictionary      |   Record |     Record |  Record | Record |
| B    | Word + 1 digit  |   Record |     Record |  Record | Record |
| C    | Word + 2 digits |   Record |     Record |  Record | Record |
| D    | Word + 4 digits |   Record |     Record |  Record | Record |

---

## 40. Prefix vs Suffix

Create two laboratory targets:

```text
2026security
security2026
```

Test:

```text
Mask + Word
```

and:

```text
Word + Mask
```

Commands:

```bash
hashcat \
    -m 1400 \
    -a 7 \
    lab.hash \
    '?d?d?d?d' \
    words.txt
```

and:

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?d?d?d?d'
```

Compare:

```text
Candidate count
Speed
Runtime
Recovery
```

---

## 41. Wordlist Size Experiment

Create:

```text
small.txt
medium.txt
large.txt
```

Test the same mask against each:

```text
?d?d
```

Record:

```text
Wordlist size
Mask size
Total keyspace
Candidate rate
Runtime
```

This shows how wordlist size affects hybrid attack cost.

---

## 42. Mask Length Experiment

Keep the same wordlist.

Test:

```text
?d
?d?d
?d?d?d
?d?d?d?d
```

Record:

```text
Mask length
Mask keyspace
Total candidates
Runtime
```

The search space increases by a factor of ten for every additional digit position.

---

# Advanced Workflow

## 43. Structured Password Research

Create laboratory targets:

```text
redteam1
redteam42
redteam2026
2026redteam
2026redteam!
```

Test the appropriate hybrid structures.

Record which attack model reaches each target.

---

## 44. Hybrid Research Directory

Recommended:

```text
hybrid-research/
├── hashes/
├── wordlists/
├── masks/
├── candidates/
├── results/
└── notes/
```

Example:

```text
hybrid-research/
├── hashes/
│   └── lab-sha256.hash
├── wordlists/
│   └── lab-words.txt
├── masks/
│   ├── numeric.txt
│   └── mixed.txt
├── candidates/
│   └── generated.txt
├── results/
│   └── experiment-001.txt
└── notes/
    └── observations.md
```

Never store real credentials or private password material.

---

## 45. Advanced Experiment

Create these laboratory targets:

```text
redteam01
redteam2026
2026redteam
security99
```

Use:

```text
words.txt
```

Run:

```text
Experiment A:
word + digit
```

```text
Experiment B:
word + two digits
```

```text
Experiment C:
word + four digits
```

```text
Experiment D:
four digits + word
```

Compare:

```text
Keyspace
Candidate rate
Runtime
Recovery
```

---

## 46. Hybrid + Rules Experiment

Generate:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r /usr/share/hashcat/rules/best64.rule \
    --stdout > transformed.txt
```

Count:

```bash
wc -l transformed.txt
```

Then:

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    transformed.txt \
    '?d?d'
```

Compare this with the original wordlist + mask attack.

---

# Troubleshooting

## 47. No Recovery

Check:

```text
Hash mode
Hash value
Wordlist
Mask
Word + mask order
Password structure
```

Preview candidates:

```bash
hashcat \
    -a 6 \
    words.txt \
    '?d?d' \
    --stdout
```

Look for the expected laboratory password.

---

## 48. Wrong Attack Direction

For:

```text
security2026
```

use:

```text
-a 6
```

For:

```text
2026security
```

use:

```text
-a 7
```

---

## 49. Too Many Candidates

Reduce:

```text
Wordlist size
Mask length
Character set
Rule expansion
```

Use verified structural information where available.

---

## 50. Verify the Hash

Make sure the generated laboratory hash matches the intended password:

```bash
printf '%s' 'YOUR-LAB-PASSWORD' | sha256sum
```

Compare it with:

```bash
cat lab.hash
```

---

# Quick Reference

## Word + Digit

```bash
hashcat -m 1400 -a 6 lab.hash words.txt '?d'
```

## Word + Two Digits

```bash
hashcat -m 1400 -a 6 lab.hash words.txt '?d?d'
```

## Word + Four Digits

```bash
hashcat -m 1400 -a 6 lab.hash words.txt '?d?d?d?d'
```

## Digit + Word

```bash
hashcat -m 1400 -a 7 lab.hash '?d' words.txt
```

## Four Digits + Word

```bash
hashcat -m 1400 -a 7 lab.hash '?d?d?d?d' words.txt
```

## Custom Charset

```bash
hashcat \
    -m 1400 \
    -a 6 \
    -1 '123456' \
    lab.hash \
    words.txt \
    '?1?1'
```

## Preview Candidates

```bash
hashcat -a 6 words.txt '?d?d' --stdout
```

## Save Candidates

```bash
hashcat -a 6 words.txt '?d?d' --stdout > candidates.txt
```

## Count Candidates

```bash
wc -l candidates.txt
```

## Show Recovered Result

```bash
hashcat -m 1400 lab.hash --show
```

## Benchmark

```bash
hashcat -b
```

---

# Research Checklist

```text
[ ] Authorized laboratory target
[ ] Disposable password
[ ] Hash format identified
[ ] Wordlist documented
[ ] Mask documented
[ ] Attack direction documented
[ ] Keyspace calculated
[ ] Candidate count measured
[ ] Hardware recorded
[ ] Hashcat version recorded
[ ] Candidate rate recorded
[ ] Runtime recorded
[ ] Recovery verified
[ ] Results preserved
[ ] No real credentials included
```

---

# Findings

A useful hybrid-attack experiment should demonstrate:

```text
Wordlist
    +
Mask
    ↓
Structured Candidate Space
```

Record:

```text
Wordlist size
Mask
Mask keyspace
Total candidate space
Candidate rate
Runtime
Recovery result
```

Hybrid attacks are most useful when the password contains a recognizable dictionary component combined with a predictable prefix or suffix. The smaller and more accurate the search space, the more efficient the offline recovery experiment becomes.
