# Brute-Force Attacks

Practical research guide to exhaustive password guessing against authorized offline hashes and disposable laboratory credentials.

Brute force is useful when there is little information about the password structure and the search space is small enough to test.

---

## 1. Attack Model

```text
Defined Character Set
        ↓
Generate Candidate
        ↓
Hash Candidate
        ↓
Compare With Target
        ↓
Match?
   ┌────┴────┐
  Yes       No
   ↓         ↓
Recover    Next Candidate
```

The attack is performed locally against authorized material.

---

## 2. Lab Setup

Install:

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

## 3. Create a Laboratory Password

Use a deliberately short password so the experiment finishes quickly.

Example:

```text
Ab12
```

Generate a SHA-256 hash:

```bash
printf '%s' 'Ab12' | sha256sum
```

Save the resulting hash:

```bash
echo '<sha256-hash>' > lab.hash
```

Never use a real password for this experiment.

---

## 4. Hashcat Brute Force

Hashcat attack mode `3` is the mask/brute-force mode.

For a four-digit laboratory password:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?d?d?d?d'
```

Here:

```text
?d = digit
```

The command tests every four-digit combination.

---

## 5. Character Sets

Hashcat built-in character classes include:

```text
?l  lowercase
?u  uppercase
?d  digits
?s  special characters
?a  all built-in characters
?b  raw bytes
```

Examples:

```text
?d?d?d?d
```

Four digits.

```text
?l?l?l?l
```

Four lowercase letters.

```text
?u?l?l?d
```

Uppercase + lowercase + lowercase + digit.

---

## 6. Custom Character Set

Define a custom set:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    -1 '?l?d' \
    lab.hash \
    '?1?1?1?1'
```

This restricts each position to lowercase letters and digits.

Custom sets are useful when the password format is known.

---

## 7. Known Prefix

Suppose the laboratory password is:

```text
Lab1234
```

If the prefix is known:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    'Lab?d?d?d?d'
```

Only the unknown portion is searched.

This is substantially more efficient than testing every possible seven-character password.

---

## 8. Known Suffix

If the laboratory password structure is:

```text
password + four digits
```

use:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    'password?d?d?d?d'
```

---

## 9. Known Length

If you know the password is exactly six lowercase characters:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?l?l?l?l?l?l'
```

If it is exactly six digits:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?d?d?d?d?d?d'
```

---

## 10. Mixed Character Password

For:

```text
Uppercase
Lowercase
Lowercase
Digit
Digit
```

use:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?u?l?l?d?d'
```

This tests only candidates matching the specified structure.

---

## 11. John the Ripper

John can also perform exhaustive searches.

Check available incremental modes:

```bash
john --list=incremental
```

Run:

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

## 12. John Character Settings

John's incremental modes are configured according to the installed John configuration.

Inspect:

```bash
john --config
```

Available behavior can vary between John installations.

For controlled research, record the exact John version and configuration used.

---

## 13. Hashcat Status

Run:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?d?d?d?d' \
    --status
```

Useful information includes:

```text
Candidates
Speed
Progress
Time remaining
Hardware utilization
```

---

## 14. Show Recovered Password

Hashcat:

```bash
hashcat \
    -m 1400 \
    lab.hash \
    --show
```

John:

```bash
john --show lab.hash
```

Verify the recovered laboratory password independently.

---

## 15. Keyspace

For a character set containing `C` characters and a password length of `L`:

```text
Keyspace = C^L
```

Examples:

```text
10 characters × 4 positions
= 10^4
= 10,000 candidates
```

Lowercase English letters:

```text
26^4
= 456,976 candidates
```

Digits:

```text
10^6
= 1,000,000 candidates
```

The keyspace grows rapidly as length increases.

---

## 16. Character-Set Comparison

For length 6:

```text
Digits:
10^6

Lowercase:
26^6

Uppercase:
26^6

Letters + digits:
62^6

Letters + digits + common symbols:
larger still
```

This demonstrates why character-set assumptions matter.

---

## 17. Password Length Experiment

Create separate laboratory hashes for:

```text
A → 4 characters
B → 5 characters
C → 6 characters
D → 7 characters
```

Use the same character set.

Record:

```text
Length
Keyspace
Measured speed
Runtime
Recovered
```

This demonstrates the effect of password length on exhaustive search.

---

## 18. Character-Set Experiment

Keep length constant.

Test:

```text
4 digits
4 lowercase
4 uppercase
4 lowercase + digits
4 mixed characters
```

Record:

```text
Character set
Keyspace
Measured speed
Runtime
```

The purpose is to isolate the effect of character-set size.

---

## 19. Known Structure Experiment

Create passwords such as:

```text
Lab1234
Lab5678
Test2026
Red1234
```

Compare two searches:

### Full search

```text
?l?l?l?l?l?l?l
```

### Known-prefix search

```text
Lab?d?d?d?d
```

Record the difference in search space and runtime.

---

## 20. Mask Position Matters

These are different search spaces:

```text
?d?d?d?d
```

and:

```text
?l?l?l?l
```

Likewise:

```text
?u?l?l?d
```

is much smaller than:

```text
?a?a?a?a
```

Do not use a larger character set when the experiment already provides reliable structural information.

---

## 21. Custom Character Set Research

Create a controlled character set:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    -1 'abc123' \
    lab.hash \
    '?1?1?1?1'
```

The four positions use only:

```text
a
b
c
1
2
3
```

Keyspace:

```text
6^4
```

This is useful for demonstrating how reducing the candidate alphabet changes search cost.

---

## 22. Multiple Custom Character Sets

Example:

```bash
hashcat \
    -m 1400 \
    -1 'ABC' \
    -2 'xyz' \
    -3 '123' \
    -a 3 \
    lab.hash \
    '?1?2?2?3'
```

Each position now has a different allowed set.

This is useful when researching structured passwords.

---

## 23. Mask Files

For repeatable research, store masks:

```bash
cat > masks.txt <<'EOF'
?d?d?d?d
?l?l?l?l
?u?l?l?d
?u?l?l?d?d
EOF
```

Review:

```bash
cat masks.txt
```

Use each mask as a separate experiment.

---

## 24. Incremental Search

A brute-force study can progress through increasing lengths:

```text
Length 1
   ↓
Length 2
   ↓
Length 3
   ↓
Length 4
   ↓
Length 5
```

This makes the growth of the search space visible.

For example:

```bash
hashcat -m 1400 -a 3 lab.hash '?d'
hashcat -m 1400 -a 3 lab.hash '?d?d'
hashcat -m 1400 -a 3 lab.hash '?d?d?d'
hashcat -m 1400 -a 3 lab.hash '?d?d?d?d'
```

Use short laboratory passwords so the experiments remain manageable.

---

## 25. Candidate Position

Suppose the correct password appears near the end of the candidate space.

The attack may have to process almost the entire space.

Record:

```text
Total keyspace
Recovered candidate
Candidate position
Candidates tested
Percentage searched
Runtime
```

This is more useful for research than simply reporting "password recovered."

---

## 26. Search Coverage

Calculate:

```text
Coverage =
Candidates Tested / Total Keyspace
```

Example:

```text
Total = 100,000
Tested = 25,000
```

Coverage:

```text
25%
```

Record this for interrupted experiments.

---

## 27. Expected vs Actual Runtime

A theoretical estimate can be calculated from:

```text
Time ≈ Keyspace / Candidate Rate
```

But actual runtime can differ because of:

```text
Hardware
Hash/KDF cost
Thermal throttling
Driver behavior
Tool overhead
Candidate generation
System load
```

Always report measured values separately from theoretical estimates.

---

## 28. Benchmark Hardware

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
Driver
Operating system
John version
Hashcat version
```

---

## 29. Hash Algorithm Matters

The same brute-force keyspace can have very different recovery times depending on the target algorithm.

Compare laboratory hashes using:

```text
Fast hash
Slow password hash
Memory-hard KDF
```

Record:

```text
Algorithm
Configuration
Candidate rate
Runtime
```

This is an important part of password-security research.

---

## 30. Dictionary vs Brute Force

Run a dictionary attack first:

```bash
john \
    --format=Raw-SHA256 \
    --wordlist=passwords.txt \
    lab.hash
```

Then run the appropriate brute-force experiment.

Compare:

```text
Attack
Candidates
Speed
Runtime
Result
```

A dictionary can find a predictable password much earlier than exhaustive search.

---

## 31. Brute Force vs Mask Attack

A full brute-force search:

```text
?a?a?a?a?a?a
```

tests a very broad character space.

A structured mask:

```text
Lab?d?d?d?d
```

tests only candidates matching the known structure.

The second approach can be dramatically smaller.

---

## 32. GPU vs CPU

Run the same laboratory experiment using:

```text
CPU
GPU
```

Record:

```text
Hardware
Algorithm
Mask
Speed
Runtime
```

Do not compare different algorithms when evaluating hardware performance.

---

## 33. Thermal Throttling

Long experiments can change performance as hardware heats up.

Record:

```text
Start speed
Average speed
End speed
Temperature
Runtime
```

For reproducibility, document whether the system was under sustained load.

---

## 34. Pause and Resume

Hashcat can save its state.

Use:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?d?d?d?d'
```

Stop safely when required.

Restore the session using the appropriate Hashcat session mechanism:

```bash
hashcat --restore
```

List or manage named sessions according to your installed version.

---

## 35. John Sessions

Create:

```bash
john \
    --session=bruteforce-lab \
    --format=Raw-SHA256 \
    --incremental \
    lab.hash
```

Status:

```bash
john --status=bruteforce-lab
```

Restore:

```bash
john --restore=bruteforce-lab
```

---

## 36. Large Keyspaces

Do not start with an enormous search space just because the hardware can process it.

First determine:

```text
Known length
Known characters
Known prefix
Known suffix
Likely structure
Algorithm
Hardware
```

Then build the smallest defensible search space.

---

## 37. Practical Experiment

Create a laboratory password:

```text
a7B2
```

Generate its hash:

```bash
printf '%s' 'a7B2' | sha256sum
```

Save:

```bash
echo '<hash>' > lab.hash
```

Run:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?a?a?a?a'
```

Check:

```bash
hashcat \
    -m 1400 \
    lab.hash \
    --show
```

Record:

```text
Password
Keyspace
Speed
Runtime
Hardware
```

---

## 38. Controlled Structure Experiment

Laboratory password:

```text
Lab2026
```

Full seven-character search:

```text
?a?a?a?a?a?a?a
```

Known-prefix search:

```text
Lab?d?d?d?d
```

Compare:

```text
Keyspace
Speed
Candidates tested
Runtime
```

The experiment demonstrates why knowledge about password structure can dramatically reduce the search space.

---

## 39. Advanced Experiment

Create five laboratory hashes:

```text
BF-01 → 4 digits
BF-02 → 5 digits
BF-03 → 6 lowercase
BF-04 → 6 lowercase + digits
BF-05 → known prefix + digits
```

Run the appropriate masks.

Record:

```text
Experiment
Algorithm
Password structure
Keyspace
Hardware
Speed
Candidates tested
Runtime
Result
```

---

## 40. Research Table

Use a table such as:

| Experiment | Structure           | Keyspace |  Speed | Runtime | Result |
| ---------- | ------------------- | -------: | -----: | ------: | ------ |
| BF-01      | 4 digits            |   Record | Record |  Record | Record |
| BF-02      | 5 digits            |   Record | Record |  Record | Record |
| BF-03      | 6 lowercase         |   Record | Record |  Record | Record |
| BF-04      | 6 lowercase+digits  |   Record | Record |  Record | Record |
| BF-05      | Known prefix+digits |   Record | Record |  Record | Record |

Use your actual measurements.

---

## 41. Troubleshooting

### No Recovery

Check:

```text
Hash format
Hash value
Character set
Password length
Mask
Attack mode
```

---

### Search Is Too Large

Reduce the search space using verified information:

```text
Known length
Known prefix
Known suffix
Known character classes
Known structure
```

Do not invent assumptions simply to make an experiment succeed.

---

### Speed Is Lower Than Expected

Check:

```text
GPU utilization
CPU utilization
Thermal throttling
Driver
Hash/KDF type
System load
```

Benchmark first:

```bash
hashcat -b
```

or:

```bash
john --test
```

---

## 42. Quick Reference

### Four Digits

```bash
hashcat -m 1400 -a 3 lab.hash '?d?d?d?d'
```

### Four Lowercase

```bash
hashcat -m 1400 -a 3 lab.hash '?l?l?l?l'
```

### Uppercase + Lowercase + Digits

```bash
hashcat -m 1400 -a 3 lab.hash '?u?l?l?d'
```

### Known Prefix

```bash
hashcat -m 1400 -a 3 lab.hash 'Lab?d?d?d?d'
```

### Custom Character Set

```bash
hashcat \
    -m 1400 \
    -a 3 \
    -1 'abc123' \
    lab.hash \
    '?1?1?1?1'
```

### John Incremental

```bash
john \
    --format=Raw-SHA256 \
    --incremental \
    lab.hash
```

### John Status

```bash
john --status
```

### Hashcat Benchmark

```bash
hashcat -b
```

### John Benchmark

```bash
john --test
```

---

## 43. Research Checklist

```text
[ ] Authorized laboratory material used
[ ] Disposable password created
[ ] Hash generated
[ ] Hash format verified
[ ] Character set documented
[ ] Password length documented
[ ] Keyspace calculated
[ ] Mask documented
[ ] Hardware recorded
[ ] Benchmark recorded
[ ] Candidate rate recorded
[ ] Runtime recorded
[ ] Coverage recorded
[ ] Recovery verified
[ ] Results preserved
[ ] No real credentials used
```

---

## 44. Key Findings to Document

A useful brute-force experiment should answer:

```text
What was the character set?
What was the password length?
How large was the keyspace?
What was the measured candidate rate?
How much of the keyspace was searched?
Was the password recovered?
How long did recovery take?
How did a known structure change the search?
How did the hashing/KDF algorithm affect performance?
```

The purpose is to produce **measurable and reproducible research**, not simply demonstrate that a password can be guessed.
