# Mask Attacks

Practical research guide to mask-based password recovery using Hashcat against authorized offline hashes.

A mask attack is useful when some information about the password structure is known. Instead of searching every possible character combination, the mask defines exactly what each position can contain.

---

## 1. Attack Model

```text
Known Password Structure
          ↓
Define Mask
          ↓
Generate Candidates
          ↓
Hash Candidate
          ↓
Compare With Target
          ↓
Match / Continue
```

Example:

```text
Lab2026
```

Known structure:

```text
Lab + 4 digits
```

Mask:

```text
Lab?d?d?d?d
```

Only the unknown positions are searched.

---

## 2. Lab Setup

Install Hashcat:

```bash
sudo apt update
sudo apt install hashcat
```

Verify:

```bash
hashcat --version
```

Check the available help:

```bash
hashcat --help
```

---

## 3. Create a Laboratory Password

Use a short disposable password.

Example:

```text
Lab27
```

Generate its SHA-256 hash:

```bash
printf '%s' 'Lab27' | sha256sum
```

Save the generated hash:

```bash
echo '<sha256-hash>' > lab.hash
```

Do not use real credentials.

---

## 4. Hashcat Mask Mode

Hashcat uses attack mode `3` for mask attacks.

Basic structure:

```bash
hashcat -m <hash-mode> -a 3 <hash-file> <mask>
```

For SHA-256:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?l?l?l?l'
```

---

## 5. Built-In Character Sets

Hashcat provides predefined character classes:

```text
?l  lowercase letters
?u  uppercase letters
?d  digits
?h  hexadecimal characters
?H  uppercase hexadecimal characters
?s  special characters
?a  all built-in character sets
?b  8-bit bytes
```

Examples:

```text
?d
```

One digit.

```text
?l
```

One lowercase letter.

```text
?u
```

One uppercase letter.

```text
?s
```

One special character.

---

## 6. Four-Digit Mask

For a four-digit laboratory password:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?d?d?d?d'
```

Search space:

```text
10^4
```

or:

```text
10,000 candidates
```

---

## 7. Six-Digit Mask

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?d?d?d?d?d?d'
```

Search space:

```text
10^6
```

or:

```text
1,000,000 candidates
```

---

## 8. Lowercase Mask

Four lowercase characters:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?l?l?l?l'
```

Search space:

```text
26^4
```

---

## 9. Mixed Mask

Suppose the laboratory password follows:

```text
Uppercase
Lowercase
Lowercase
Digit
Digit
```

Use:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?u?l?l?d?d'
```

This is much smaller than allowing every character class at every position.

---

## 10. Known Prefix

Laboratory password:

```text
Lab27
```

Known:

```text
Lab
```

Unknown:

```text
27
```

Mask:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    'Lab?d?d'
```

Only the two digits are generated.

---

## 11. Known Suffix

Suppose the structure is:

```text
two digits + Lab
```

Use:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?d?dLab'
```

The fixed portion does not contribute to the unknown search space.

---

## 12. Known Prefix and Suffix

Suppose:

```text
Lab + 3 digits + !
```

Mask:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    'Lab?d?d?d!'
```

This is one of the main advantages of masks: known information can be incorporated directly.

---

## 13. Custom Character Sets

Hashcat allows custom character sets using `-1` through `-4`.

Example:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    -1 'abc123' \
    lab.hash \
    '?1?1?1?1'
```

Each `?1` represents one character from:

```text
a
b
c
1
2
3
```

---

## 14. Multiple Custom Character Sets

Example:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    -1 'ABC' \
    -2 'xyz' \
    -3 '123' \
    lab.hash \
    '?1?2?2?3'
```

The mask means:

```text
Position 1 → ABC
Position 2 → xyz
Position 3 → xyz
Position 4 → 123
```

This is useful when the password structure is known.

---

## 15. Custom Charset + Fixed Prefix

Example structure:

```text
Lab + one letter + two digits
```

Command:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    -1 '?l' \
    lab.hash \
    'Lab?1?d?d'
```

The custom charset can also be used for more specialized research patterns.

---

## 16. Character-Set Reduction

Compare:

```text
?a?a?a?a
```

with:

```text
?l?l?l?l
```

and:

```text
?d?d?d?d
```

The search spaces are dramatically different.

For example:

```text
Digits:
10^4

Lowercase:
26^4

All built-in characters:
much larger
```

Use the smallest justified character set.

---

## 17. Mask Position

These are different:

```text
?u?l?l?d
```

and:

```text
?l?u?d?l
```

The position of each character class matters.

If password structure is known, place the character classes exactly where they belong.

---

## 18. Password Pattern Research

Create laboratory passwords such as:

```text
Red123
Lab2026
Test42
Cyber007
```

Identify their patterns:

```text
Red + digits
Lab + digits
Test + digits
Cyber + digits
```

Then construct masks matching each structure.

Do not infer patterns from private credentials belonging to other people.

---

## 19. Mask Ordering

Candidate order affects when a password is recovered.

For a simple numeric mask:

```text
?d?d?d?d
```

the search covers the available combinations systematically.

Record:

```text
Candidate position
Total keyspace
Candidates tested
Recovery percentage
Runtime
```

This provides more useful research data than simply recording whether the password was found.

---

## 20. Keyspace Calculation

For a mask containing positions with character-set sizes:

```text
C1 × C2 × C3 × ... × Cn
```

Example:

```text
?u?l?l?d
```

Character counts:

```text
26 × 26 × 26 × 10
```

Therefore:

```text
175,760 candidates
```

The exact keyspace should be calculated before beginning a large experiment.

---

## 21. Keyspace Examples

### Four digits

```text
10^4
= 10,000
```

### Six digits

```text
10^6
= 1,000,000
```

### Four lowercase letters

```text
26^4
= 456,976
```

### Uppercase + lowercase + digits

For six positions:

```text
62^6
```

This demonstrates how quickly the search space grows.

---

## 22. Known Structure vs Full Search

Consider:

```text
Lab2026
```

A broad seven-character search:

```text
?a?a?a?a?a?a?a
```

is enormous.

A known structure:

```text
Lab?d?d?d?d
```

contains only:

```text
10^4
```

unknown combinations.

This is why reconnaissance of password structure can have a major effect on offline recovery.

---

## 23. Hashcat Status

Run:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?d?d?d?d' \
    --status
```

Monitor:

```text
Speed
Progress
Candidates
Estimated time
Hardware utilization
```

Record the measurements.

---

## 24. Hashcat Benchmark

Run:

```bash
hashcat -b
```

Record:

```text
GPU
CPU
Driver
Hash algorithm
Measured speed
```

Benchmark data should be collected on the same system used for the experiment.

---

## 25. Session Management

Create a named session:

```bash
hashcat \
    --session=mask-lab \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?d?d?d?d'
```

Restore:

```bash
hashcat --session=mask-lab --restore
```

Or use:

```bash
hashcat --restore
```

depending on how the session was started.

---

## 26. Workload Management

For long experiments, Hashcat supports workload controls.

Check:

```bash
hashcat --help
```

Options can vary between versions.

Record the relevant settings in the experiment log so results remain reproducible.

---

## 27. Incremental Length Testing

Instead of immediately testing a large length:

```text
4 characters
↓
5 characters
↓
6 characters
↓
7 characters
```

For digits:

```bash
hashcat -m 1400 -a 3 lab.hash '?d?d?d?d'
hashcat -m 1400 -a 3 lab.hash '?d?d?d?d?d'
hashcat -m 1400 -a 3 lab.hash '?d?d?d?d?d?d'
```

Record the keyspace and runtime for every length.

---

## 28. Compare Character Sets

Create separate laboratory experiments:

```text
MASK-01 → ?d?d?d?d
MASK-02 → ?l?l?l?l
MASK-03 → ?u?l?l?d
MASK-04 → ?l?l?d?d
```

Record:

```text
Mask
Keyspace
Speed
Runtime
Result
```

This isolates the effect of different character classes.

---

## 29. Compare Full vs Structured Masks

Experiment:

```text
Password:
Lab2026
```

Test:

```text
Full search
```

against:

```text
Lab?d?d?d?d
```

Record:

```text
Search space
Candidates tested
Runtime
Result
```

This is a useful demonstration of structural information reducing computational cost.

---

## 30. Benchmark Different Hashes

Use separate laboratory hashes with identical password structures.

Compare:

```text
Fast hash
Slow password hash
Memory-hard KDF
```

Keep the password structure constant.

Record:

```text
Algorithm
Configuration
Candidate rate
Runtime
Hardware
```

This demonstrates why password-storage algorithms matter.

---

## 31. GPU vs CPU

Run the same controlled mask against the same laboratory hash.

Record:

```text
CPU:
Speed:
Runtime:

GPU:
Speed:
Runtime:
```

Do not change the mask or hash between the tests.

---

## 32. Thermal Effects

Long cracking jobs can cause hardware temperature and performance to change.

Record:

```text
Initial speed
Average speed
Final speed
Temperature
Total runtime
```

If performance drops during the experiment, document it.

---

## 33. Candidate Coverage

If an experiment stops before completion:

```text
Coverage =
Candidates Tested / Total Keyspace
```

Example:

```text
Total keyspace = 1,000,000
Candidates tested = 250,000
```

Coverage:

```text
25%
```

Record this rather than describing the experiment simply as "failed."

---

## 34. Estimated Runtime

Approximate full-search time:

```text
Runtime ≈ Keyspace / Candidate Rate
```

Example:

```text
Keyspace:
1,000,000

Measured rate:
500,000 candidates/sec
```

Approximate time:

```text
2 seconds
```

Actual results can differ because of system conditions and hash-processing overhead.

---

## 35. Mask Files

Keep masks in a dedicated research file:

```bash
cat > masks.txt <<'EOF'
?d?d?d?d
?d?d?d?d?d
?l?l?l?l
?u?l?l?d
EOF
```

Inspect:

```bash
cat masks.txt
```

This makes experiments easier to reproduce.

---

## 36. Research Directory

A clean structure:

```text
mask-research/
├── hashes/
├── masks/
├── results/
└── notes/
```

Example:

```text
mask-research/
├── hashes/
│   └── lab-sha256.hash
├── masks/
│   ├── numeric.txt
│   └── structured.txt
├── results/
│   └── experiment-001.txt
└── notes/
    └── observations.md
```

Do not commit real credentials or private hashes.

---

## 37. Experimental Workflow

```text
Create Disposable Password
          ↓
Generate Hash
          ↓
Identify Hash Format
          ↓
Determine Known Structure
          ↓
Define Character Sets
          ↓
Build Mask
          ↓
Calculate Keyspace
          ↓
Benchmark Hardware
          ↓
Run Mask Attack
          ↓
Record Results
          ↓
Verify Recovery
          ↓
Compare Experiments
```

---

## 38. Practical Experiment

Create:

```text
Lab42
```

Generate:

```bash
printf '%s' 'Lab42' | sha256sum
```

Save the hash:

```bash
echo '<sha256-hash>' > lab.hash
```

Run:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    'Lab?d?d'
```

Show the result:

```bash
hashcat \
    -m 1400 \
    lab.hash \
    --show
```

Record:

```text
Mask: Lab?d?d
Keyspace: 100
Speed: measured
Runtime: measured
Result: recovered / not recovered
```

---

## 39. Advanced Structured Experiment

Create a disposable password:

```text
Test2026!
```

Known structure:

```text
Test + 4 digits + !
```

Mask:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    'Test?d?d?d?d!'
```

Compare it against a much broader laboratory mask and document the difference in keyspace.

---

## 40. Custom Alphabet Experiment

Define:

```text
a
b
c
1
2
3
```

Command:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    -1 'abc123' \
    lab.hash \
    '?1?1?1?1'
```

Keyspace:

```text
6^4
```

Record:

```text
Alphabet
Length
Keyspace
Speed
Runtime
Result
```

---

## 41. Mask Attack Research Matrix

| Experiment | Mask           | Keyspace |  Speed | Runtime | Result |
| ---------- | -------------- | -------: | -----: | ------: | ------ |
| MASK-01    | `?d?d?d?d`     |   Record | Record |  Record | Record |
| MASK-02    | `?l?l?l?l`     |   Record | Record |  Record | Record |
| MASK-03    | `?u?l?l?d`     |   Record | Record |  Record | Record |
| MASK-04    | `Lab?d?d`      |   Record | Record |  Record | Record |
| MASK-05    | Custom charset |   Record | Record |  Record | Record |

Use actual measurements from your laboratory.

---

## 42. Troubleshooting

### No Recovery

Check:

```text
Hash format
Hash value
Mask
Password length
Character classes
Fixed characters
```

A single incorrect character in the mask can exclude the correct password entirely.

---

### Search Space Too Large

Reduce it using verified structural information:

```text
Known prefix
Known suffix
Known length
Known character classes
Known positions
```

Do not make unsupported assumptions simply to reduce the search.

---

### Low Speed

Check:

```text
GPU utilization
CPU utilization
Driver
Thermal throttling
Hash algorithm
System load
```

Benchmark:

```bash
hashcat -b
```

---

### Incorrect Hash Mode

Inspect available hash information:

```bash
hashcat --example-hashes
```

Use the correct mode for the actual laboratory hash format.

Do not assume `1400` applies to every SHA-256-related format.

---

## 43. Quick Reference

### Four Digits

```bash
hashcat -m 1400 -a 3 lab.hash '?d?d?d?d'
```

### Four Lowercase

```bash
hashcat -m 1400 -a 3 lab.hash '?l?l?l?l'
```

### Mixed Pattern

```bash
hashcat -m 1400 -a 3 lab.hash '?u?l?l?d'
```

### Known Prefix

```bash
hashcat -m 1400 -a 3 lab.hash 'Lab?d?d'
```

### Known Prefix + Suffix

```bash
hashcat -m 1400 -a 3 lab.hash 'Lab?d?d!'
```

### Custom Charset

```bash
hashcat \
    -m 1400 \
    -a 3 \
    -1 'abc123' \
    lab.hash \
    '?1?1?1?1'
```

### Benchmark

```bash
hashcat -b
```

### Show Result

```bash
hashcat -m 1400 lab.hash --show
```

---

## 44. Research Checklist

```text
[ ] Authorized laboratory target
[ ] Disposable password
[ ] Hash format identified
[ ] Password structure documented
[ ] Character sets documented
[ ] Mask documented
[ ] Keyspace calculated
[ ] Hardware recorded
[ ] Benchmark completed
[ ] Candidate rate recorded
[ ] Runtime recorded
[ ] Coverage recorded
[ ] Result verified
[ ] Experiment reproducible
[ ] No real credentials included
```

---

## 45. Findings

Every mask experiment should answer:

```text
What structure was known?
What mask was selected?
How large was the keyspace?
What character sets were used?
What was the measured candidate rate?
How much of the keyspace was searched?
Was the password recovered?
How much did structural knowledge reduce the search?
How did the hash/KDF affect recovery time?
```

The objective is to demonstrate how **password structure, character-set size, keyspace, hardware, and password-storage algorithms affect offline recovery cost**.
