# Combinator Attacks

Practical guide to combining two wordlists into password candidates with Hashcat.

A combinator attack joins entries from two wordlists.

```text
Wordlist A + Wordlist B
        ↓
    Candidates
        ↓
      Hash
        ↓
    Comparison
```

Examples:

```text
red + team
redteam + 2026
summer + 2026
admin + portal
```

All examples below are intended for disposable laboratory hashes and systems you are authorized to test.

---

## 1. Attack Mode

Hashcat uses:

```text
-a 1
```

for combinator attacks.

Basic syntax:

```bash
hashcat \
    -m <hash-mode> \
    -a 1 \
    <hash-file> \
    <wordlist-left> \
    <wordlist-right>
```

The first wordlist supplies the left side and the second supplies the right side.

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

Check supported options:

```bash
hashcat --help
```

---

## 3. Create the First Wordlist

```bash
cat > words-left.txt <<'EOF'
red
blue
green
admin
security
cyber
EOF
```

Check:

```bash
cat words-left.txt
```

---

## 4. Create the Second Wordlist

```bash
cat > words-right.txt <<'EOF'
team
lab
2024
2025
2026
portal
EOF
```

Check:

```bash
cat words-right.txt
```

---

## 5. Count Both Lists

```bash
wc -l words-left.txt
wc -l words-right.txt
```

If the first list contains:

```text
6
```

and the second contains:

```text
6
```

the theoretical combination count is:

```text
6 × 6 = 36
```

---

# Candidate Generation

## 6. Preview Combinations

Before attacking a hash, inspect the candidates.

```bash
hashcat \
    -a 1 \
    words-left.txt \
    words-right.txt \
    --stdout
```

Example output:

```text
redteam
redlab
red2024
red2025
red2026
redportal
blueteam
bluelab
...
```

This is one of the most important steps when developing an attack.

---

## 7. Save Candidates

```bash
hashcat \
    -a 1 \
    words-left.txt \
    words-right.txt \
    --stdout > combinations.txt
```

Check:

```bash
wc -l combinations.txt
```

View:

```bash
head combinations.txt
```

---

## 8. Inspect the End

```bash
tail combinations.txt
```

This confirms the complete generated range.

---

# Basic Hash Recovery

## 9. Create a Laboratory Password

For example:

```text
redteam
```

Generate SHA-256:

```bash
printf '%s' 'redteam' | sha256sum
```

Save the hash:

```bash
echo '<sha256-hash>' > lab.hash
```

---

## 10. Run the Combinator Attack

```bash
hashcat \
    -m 1400 \
    -a 1 \
    lab.hash \
    words-left.txt \
    words-right.txt
```

Show recovered result:

```bash
hashcat \
    -m 1400 \
    lab.hash \
    --show
```

---

# Wordlist Ordering

## 11. Left + Right

Suppose:

```text
words-left.txt
red
```

and:

```text
words-right.txt
team
```

The result is:

```text
redteam
```

Command:

```bash
hashcat \
    -a 1 \
    words-left.txt \
    words-right.txt \
    --stdout
```

---

## 12. Reverse the Order

The reverse combination:

```text
teamred
```

requires:

```bash
hashcat \
    -a 1 \
    words-right.txt \
    words-left.txt \
    --stdout
```

Order matters.

These are different candidate spaces:

```text
A + B
```

versus:

```text
B + A
```

---

# Keyspace

## 13. Basic Formula

For two wordlists:

```text
Total Candidates =
Size(A) × Size(B)
```

Example:

```text
A = 1,000 words
B = 500 words
```

Then:

```text
1,000 × 500
= 500,000 candidates
```

---

## 14. Calculate With Shell

```bash
wc -l words-left.txt
wc -l words-right.txt
```

For a quick calculation:

```bash
echo "$(( $(wc -l < words-left.txt) * $(wc -l < words-right.txt) ))"
```

---

## 15. Three-List Combinations

A standard Hashcat combinator attack joins two wordlists at a time.

For three components:

```text
word + word + number
```

prepare the appropriate candidate list or use another attack strategy.

For example, first generate:

```bash
hashcat \
    -a 1 \
    words-left.txt \
    words-right.txt \
    --stdout > first-stage.txt
```

Then combine:

```bash
hashcat \
    -a 1 \
    first-stage.txt \
    numbers.txt \
    --stdout > second-stage.txt
```

This can expand very quickly, so calculate the resulting keyspace before generating large files.

---

# Practical Experiments

## 16. Word + Word

Create:

```bash
cat > first.txt <<'EOF'
red
blue
green
EOF
```

Create:

```bash
cat > second.txt <<'EOF'
team
house
lab
EOF
```

Generate:

```bash
hashcat \
    -a 1 \
    first.txt \
    second.txt \
    --stdout
```

Expected structures:

```text
redteam
redhouse
redlab
blueteam
bluehouse
bluelab
greenteam
greenhouse
greenlab
```

---

## 17. Name + Number

Create:

```bash
cat > names.txt <<'EOF'
admin
security
redteam
operator
EOF
```

Create:

```bash
cat > numbers.txt <<'EOF'
01
02
2024
2025
2026
EOF
```

Run:

```bash
hashcat \
    -a 1 \
    names.txt \
    numbers.txt \
    --stdout
```

Candidates include:

```text
admin01
admin02
admin2024
admin2025
admin2026
security01
security02
...
```

---

## 18. Word + Symbol

Create:

```bash
cat > symbols.txt <<'EOF'
!
@
#
$
EOF
```

Run:

```bash
hashcat \
    -a 1 \
    names.txt \
    symbols.txt \
    --stdout
```

This produces candidates such as:

```text
admin!
admin@
admin#
admin$
```

---

## 19. Prefix Wordlist + Suffix Wordlist

Create:

```bash
cat > prefixes.txt <<'EOF'
red
blue
cyber
EOF
```

Create:

```bash
cat > suffixes.txt <<'EOF'
team
security
lab
EOF
```

Run:

```bash
hashcat \
    -a 1 \
    prefixes.txt \
    suffixes.txt \
    --stdout
```

---

# Duplicate Handling

## 20. Generate and Deduplicate

Generate:

```bash
hashcat \
    -a 1 \
    words-left.txt \
    words-right.txt \
    --stdout > combinations.txt
```

Count:

```bash
wc -l combinations.txt
```

Deduplicate:

```bash
sort -u combinations.txt > unique-combinations.txt
```

Count unique candidates:

```bash
wc -l unique-combinations.txt
```

Compare:

```text
Original candidates
Unique candidates
Duplicate count
```

---

## 21. Find Duplicate Candidates

```bash
sort combinations.txt | uniq -d
```

This shows candidates generated more than once.

Count them:

```bash
sort combinations.txt | uniq -d | wc -l
```

---

# Wordlist Preparation

## 22. Remove Empty Lines

```bash
sed -i '/^[[:space:]]*$/d' words-left.txt
sed -i '/^[[:space:]]*$/d' words-right.txt
```

---

## 23. Remove Duplicate Words

```bash
sort -u words-left.txt -o words-left.txt
sort -u words-right.txt -o words-right.txt
```

Verify:

```bash
wc -l words-left.txt
wc -l words-right.txt
```

---

## 24. Normalize a Wordlist

Inspect unusual entries:

```bash
cat -n words-left.txt
```

Look for:

```text
empty entries
leading spaces
trailing spaces
duplicate entries
unexpected characters
```

Keep a clean copy of the original research data before modifying it.

---

# Combinator Attack Against a Laboratory Hash

## 25. Complete Example

Create:

```bash
cat > first.txt <<'EOF'
password
admin
red
security
EOF
```

Create:

```bash
cat > second.txt <<'EOF'
123
2025
2026
team
EOF
```

Generate:

```bash
hashcat \
    -a 1 \
    first.txt \
    second.txt \
    --stdout
```

Create the laboratory password:

```text
red2026
```

Generate its SHA-256 hash:

```bash
printf '%s' 'red2026' | sha256sum
```

Save:

```bash
echo '<sha256-hash>' > lab.hash
```

Run:

```bash
hashcat \
    -m 1400 \
    -a 1 \
    lab.hash \
    first.txt \
    second.txt
```

Show:

```bash
hashcat \
    -m 1400 \
    lab.hash \
    --show
```

---

# Combining With Rules

## 26. Rule-Transform One List

Create transformed candidates:

```bash
hashcat \
    -a 0 \
    first.txt \
    -r /usr/share/hashcat/rules/best64.rule \
    --stdout > transformed.txt
```

Then combine:

```bash
hashcat \
    -a 1 \
    transformed.txt \
    second.txt \
    --stdout
```

This creates:

```text
Transformed word
       +
Second word
       ↓
Candidate
```

---

## 27. Estimate Expansion

Suppose:

```text
first.txt = 10,000 entries
second.txt = 1,000 entries
```

Basic combination:

```text
10,000 × 1,000
= 10,000,000
```

If rule transformation expands the first list to 50,000 candidates:

```text
50,000 × 1,000
= 50,000,000
```

This demonstrates why rule + combinator attacks can become expensive very quickly.

---

# Performance

## 28. Hashcat Benchmark

```bash
hashcat -b
```

Record the relevant hash algorithm speed.

---

## 29. Candidate Generation Benchmark

```bash
time hashcat \
    -a 1 \
    first.txt \
    second.txt \
    --stdout > /dev/null
```

Record:

```text
First list size
Second list size
Total candidates
Generation time
```

---

## 30. Recovery Benchmark

```bash
time hashcat \
    -m 1400 \
    -a 1 \
    lab.hash \
    first.txt \
    second.txt
```

Record:

```text
Hash mode
Wordlist A
Wordlist B
Keyspace
Hash rate
Runtime
Recovery
```

---

# Session Management

## 31. Named Session

```bash
hashcat \
    --session=combinator-lab \
    -m 1400 \
    -a 1 \
    lab.hash \
    first.txt \
    second.txt
```

Restore:

```bash
hashcat \
    --session=combinator-lab \
    --restore
```

---

## 32. Show Status

During an active Hashcat session:

```text
s
```

Typical status information includes:

```text
Progress
Speed
Time estimated
Candidates
Recovered
```

---

# Advanced Experiments

## 33. Small × Small

Use:

```text
100 × 100
```

Generate:

```bash
hashcat \
    -a 1 \
    small-a.txt \
    small-b.txt \
    --stdout > test.txt
```

Measure:

```bash
wc -l test.txt
```

Expected theoretical count:

```text
10,000
```

---

## 34. Small × Large

Test:

```text
100 × 10,000
```

Calculate:

```text
1,000,000 candidates
```

Measure actual output:

```bash
wc -l test.txt
```

---

## 35. Large × Large

Before attempting a large experiment:

```bash
echo "$(( $(wc -l < large-a.txt) * $(wc -l < large-b.txt) ))"
```

Do not generate massive candidate files unnecessarily.

If the keyspace is large, run Hashcat directly instead of using `--stdout` to create a huge intermediate file.

---

# Candidate Space Research

## 36. Compare Different Lists

Test:

```text
A × B
A × C
B × C
```

Record:

| Experiment | List A | List B | Candidates | Runtime | Result |
| ---------- | ------ | ------ | ---------: | ------: | ------ |
| 1          | small  | small  |     Record |  Record | Record |
| 2          | small  | medium |     Record |  Record | Record |
| 3          | medium | medium |     Record |  Record | Record |

---

## 37. Order Comparison

Run:

```bash
hashcat \
    -a 1 \
    first.txt \
    second.txt \
    --stdout > forward.txt
```

Then:

```bash
hashcat \
    -a 1 \
    second.txt \
    first.txt \
    --stdout > reverse.txt
```

Compare:

```bash
head forward.txt
head reverse.txt
```

The same two lists can produce different candidate ordering depending on which list is first.

---

# Three-Stage Research

## 38. First Stage

```bash
hashcat \
    -a 1 \
    first.txt \
    second.txt \
    --stdout > stage1.txt
```

Count:

```bash
wc -l stage1.txt
```

---

## 39. Second Stage

Create:

```bash
cat > third.txt <<'EOF'
1
2
2026
!
EOF
```

Combine:

```bash
hashcat \
    -a 1 \
    stage1.txt \
    third.txt \
    --stdout > stage2.txt
```

Count:

```bash
wc -l stage2.txt
```

---

## 40. Avoid Unnecessary Intermediate Files

For large experiments, intermediate files can consume significant disk space.

Check:

```bash
du -h stage1.txt
```

Check available disk:

```bash
df -h
```

For large keyspaces, prefer direct Hashcat attacks where possible.

---

# Research Dataset

## 41. Recommended Structure

```text
combinator-research/
├── hashes/
│   └── lab.hash
├── wordlists/
│   ├── first.txt
│   ├── second.txt
│   └── third.txt
├── candidates/
│   └── combinations.txt
├── results/
│   └── experiment-001.txt
└── notes/
    └── observations.md
```

Do not commit real passwords, credential dumps, or private hashes to a public repository.

---

# Practical Lab

## 42. Target

Use the disposable laboratory password:

```text
cyberlab2026
```

Generate:

```bash
printf '%s' 'cyberlab2026' | sha256sum
```

Save:

```bash
echo '<sha256-hash>' > lab.hash
```

Create:

```bash
cat > first.txt <<'EOF'
cyber
red
blue
security
EOF
```

Create:

```bash
cat > second.txt <<'EOF'
lab
2024
2025
2026
EOF
```

Preview:

```bash
hashcat \
    -a 1 \
    first.txt \
    second.txt \
    --stdout
```

Run:

```bash
hashcat \
    -m 1400 \
    -a 1 \
    lab.hash \
    first.txt \
    second.txt
```

Verify:

```bash
hashcat \
    -m 1400 \
    lab.hash \
    --show
```

---

# Advanced Lab

## 43. Multiple Components

Create:

```bash
cat > base.txt <<'EOF'
red
cyber
security
EOF
```

Create:

```bash
cat > topic.txt <<'EOF'
team
lab
ops
EOF
```

Create:

```bash
cat > suffix.txt <<'EOF'
01
2025
2026
!
EOF
```

Stage one:

```bash
hashcat \
    -a 1 \
    base.txt \
    topic.txt \
    --stdout > stage1.txt
```

Stage two:

```bash
hashcat \
    -a 1 \
    stage1.txt \
    suffix.txt \
    --stdout > final-candidates.txt
```

Inspect:

```bash
head final-candidates.txt
```

Count:

```bash
wc -l final-candidates.txt
```

---

# Results

## 44. Record Every Experiment

Use:

| ID  | List A          | List B      | A Size | B Size | Keyspace | Runtime | Result |
| --- | --------------- | ----------- | -----: | -----: | -------: | ------: | ------ |
| 001 | first.txt       | second.txt  | Record | Record |   Record |  Record | Record |
| 002 | small.txt       | numbers.txt | Record | Record |   Record |  Record | Record |
| 003 | transformed.txt | suffix.txt  | Record | Record |   Record |  Record | Record |

Record measured values rather than estimated performance.

---

## 45. What to Measure

For each experiment:

```text
Wordlist A size
Wordlist B size
Theoretical keyspace
Actual candidates
Unique candidates
Hash algorithm
Hashcat version
Hardware
Hash rate
Runtime
Recovery result
```

---

# Troubleshooting

## 46. Expected Password Does Not Appear

Preview:

```bash
hashcat \
    -a 1 \
    first.txt \
    second.txt \
    --stdout | grep -Fx 'EXPECTED-LAB-PASSWORD'
```

If nothing is returned, the candidate is not represented by the two lists in their current order.

---

## 47. Check Wordlist Entries

```bash
grep -n 'cyber' first.txt
grep -n '2026' second.txt
```

Check hidden characters:

```bash
cat -A first.txt
cat -A second.txt
```

---

## 48. Wrong Order

For:

```text
cyberlab
```

use:

```text
cyber + lab
```

For:

```text
labcyber
```

reverse the lists:

```bash
hashcat \
    -a 1 \
    second.txt \
    first.txt \
    --stdout
```

---

## 49. Large Keyspace

Calculate first:

```bash
echo "$(( $(wc -l < first.txt) * $(wc -l < second.txt) ))"
```

If the result is very large:

```text
Reduce list size
Remove duplicates
Use targeted wordlists
Use known structure
Avoid unnecessary stages
```

---

# Quick Reference

## Basic Combinator

```bash
hashcat \
    -a 1 \
    first.txt \
    second.txt
```

## Preview

```bash
hashcat \
    -a 1 \
    first.txt \
    second.txt \
    --stdout
```

## Save Candidates

```bash
hashcat \
    -a 1 \
    first.txt \
    second.txt \
    --stdout > combinations.txt
```

## Count

```bash
wc -l combinations.txt
```

## Unique

```bash
sort -u combinations.txt > unique.txt
```

## Hash Recovery

```bash
hashcat \
    -m 1400 \
    -a 1 \
    lab.hash \
    first.txt \
    second.txt
```

## Reverse Order

```bash
hashcat \
    -m 1400 \
    -a 1 \
    lab.hash \
    second.txt \
    first.txt
```

## Keyspace

```bash
echo "$(( $(wc -l < first.txt) * $(wc -l < second.txt) ))"
```

## Benchmark

```bash
hashcat -b
```

## Show Result

```bash
hashcat -m 1400 lab.hash --show
```

---

# Research Checklist

```text
[ ] Authorized laboratory target
[ ] Disposable password
[ ] Hash format verified
[ ] First wordlist documented
[ ] Second wordlist documented
[ ] Wordlist duplicates removed
[ ] Keyspace calculated
[ ] Candidate generation tested
[ ] Candidate ordering checked
[ ] Hashcat version recorded
[ ] Hardware recorded
[ ] Hash rate recorded
[ ] Runtime recorded
[ ] Recovery verified
[ ] Results preserved
[ ] No real credentials included
```

---

# Findings

A combinator attack explores combinations of two candidate sets:

```text
Wordlist A
    ×
Wordlist B
    ↓
Candidate Space
```

The central measurement is:

```text
|A| × |B|
```

As either wordlist grows, the candidate space grows proportionally.

For research, keep the wordlists targeted, measure the resulting keyspace, and compare different combinations rather than blindly combining large datasets.
