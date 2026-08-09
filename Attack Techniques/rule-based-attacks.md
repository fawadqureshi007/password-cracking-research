# Rule-Based Attacks

Practical guide to transforming dictionary candidates with Hashcat and John the Ripper.

Rule-based attacks start with a wordlist and modify each candidate according to defined transformations. This is useful when a password is based on a known word but has predictable changes such as capitalization, numbers, prefixes, suffixes, or substitutions.

All examples below are intended for authorized offline password-recovery labs.

---

## 1. Attack Model

```text
Wordlist
   ↓
Rule Engine
   ↓
Candidate Transformation
   ↓
Hash Candidate
   ↓
Compare With Target
   ↓
Recovered / Continue
```

Example:

```text
password
Password
password1
Password1
password!
Password2026
```

One dictionary word can therefore produce many candidates.

---

## 2. Lab Setup

Install Hashcat and John:

```bash
sudo apt update
sudo apt install hashcat john
```

Verify:

```bash
hashcat --version
john --version
```

---

## 3. Create a Laboratory Wordlist

Create:

```bash
cat > words.txt <<'EOF'
password
admin
summer
dragon
redteam
security
EOF
```

Check:

```bash
cat words.txt
```

Count entries:

```bash
wc -l words.txt
```

---

## 4. Create a Laboratory Password

Use a disposable password derived from one of the words.

Example:

```text
Redteam2026!
```

Generate a SHA-256 hash:

```bash
printf '%s' 'Redteam2026!' | sha256sum
```

Save the hash:

```bash
echo '<sha256-hash>' > lab.hash
```

---

# Hashcat Rules

## 5. Hashcat Rule Attack

Hashcat uses attack mode `0` for a wordlist attack.

Basic syntax:

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    words.txt \
    -r rules/best64.rule
```

The rule file transforms every word in the wordlist.

---

## 6. List Available Rules

On many Linux installations:

```bash
ls /usr/share/hashcat/rules/
```

Common rule files may include:

```text
best64.rule
dive.rule
rockyou-30000.rule
InsidePro-PasswordsPro.rule
```

Available files depend on the installed Hashcat package.

Inspect a rule:

```bash
head -20 /usr/share/hashcat/rules/best64.rule
```

---

## 7. Best64

Run:

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    words.txt \
    -r /usr/share/hashcat/rules/best64.rule
```

`best64.rule` applies a relatively small collection of common transformations.

It is useful as a first controlled experiment.

---

## 8. Rule Candidate Generation

Before running a large attack, inspect what a rule produces.

Hashcat can output transformed candidates using:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r /usr/share/hashcat/rules/best64.rule \
    --stdout
```

Save the output:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r /usr/share/hashcat/rules/best64.rule \
    --stdout > candidates.txt
```

Count candidates:

```bash
wc -l candidates.txt
```

This is useful for measuring rule expansion.

---

# Hashcat Rule Syntax

## 9. Basic Rule Operations

Hashcat rules operate on individual dictionary candidates.

Common operations include:

```text
:
```

No operation.

```text
l
```

Convert characters to lowercase.

```text
u
```

Convert characters to uppercase.

```text
c
```

Capitalize first character and lowercase the remainder.

```text
C
```

Lowercase first character and uppercase the remainder.

```text
t
```

Toggle case.

```text
r
```

Reverse the word.

```text
d
```

Duplicate the word.

```text
p1
```

Duplicate the word with the specified operation count where supported by the rule syntax.

Always verify the syntax against your installed Hashcat version:

```bash
hashcat --help
```

---

## 10. Append a Character

A common transformation is adding a character to the end.

Example rule:

```text
$1
```

Create:

```bash
cat > append.rule <<'EOF'
$1
EOF
```

Test:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r append.rule \
    --stdout
```

For:

```text
password
```

the output includes:

```text
password1
```

---

## 11. Append Multiple Characters

Example:

```text
$1$2$3
```

Create:

```bash
cat > suffix.rule <<'EOF'
$1$2$3
EOF
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r suffix.rule \
    --stdout
```

A candidate such as:

```text
password123
```

can be generated from:

```text
password
```

---

## 12. Prepend a Character

Example:

```text
^1
```

Create:

```bash
cat > prefix.rule <<'EOF'
^1
EOF
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r prefix.rule \
    --stdout
```

This creates candidates with `1` at the beginning.

---

## 13. Capitalization

Create:

```bash
cat > capitalize.rule <<'EOF'
c
EOF
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r capitalize.rule \
    --stdout
```

For:

```text
password
```

the transformation produces:

```text
Password
```

---

## 14. Uppercase Transformation

Rule:

```text
u
```

Create:

```bash
echo 'u' > uppercase.rule
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r uppercase.rule \
    --stdout
```

---

## 15. Lowercase Transformation

Rule:

```text
l
```

Create:

```bash
echo 'l' > lowercase.rule
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r lowercase.rule \
    --stdout
```

---

## 16. Reverse

Rule:

```text
r
```

Create:

```bash
echo 'r' > reverse.rule
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r reverse.rule \
    --stdout
```

---

## 17. Toggle Case

Rule:

```text
t
```

Create:

```bash
echo 't' > toggle.rule
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r toggle.rule \
    --stdout
```

---

## 18. Combine Transformations

Rules can contain multiple operations.

Example:

```text
c$1
```

This means:

```text
Capitalize
+
append 1
```

Create:

```bash
echo 'c$1' > capitalize-number.rule
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r capitalize-number.rule \
    --stdout
```

For:

```text
password
```

the result includes:

```text
Password1
```

---

## 19. Multiple Rules

Create:

```bash
cat > custom.rule <<'EOF'
:
c
$1
$2
$3
c$1
c$2
c$3
$!
c$!
EOF
```

Run:

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    words.txt \
    -r custom.rule
```

This tests several transformations against every word.

---

## 20. Inspect Generated Candidates

Before attacking the hash:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r custom.rule \
    --stdout
```

This lets you verify exactly what your rule set generates.

Save:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r custom.rule \
    --stdout > generated.txt
```

Count:

```bash
wc -l generated.txt
```

---

## 21. Avoid Duplicate Candidates

Large rule sets can generate the same candidate more than once.

Deduplicate:

```bash
sort -u generated.txt > unique.txt
```

Count:

```bash
wc -l generated.txt
wc -l unique.txt
```

Compare the difference.

This is useful when researching rule efficiency.

---

# John the Ripper Rules

## 22. John Wordlist Mode

Basic command:

```bash
john \
    --format=Raw-SHA256 \
    --wordlist=words.txt \
    lab.hash
```

Show results:

```bash
john --show lab.hash
```

---

## 23. John Rules

John rules are configured in its configuration file.

Locate the configuration:

```bash
john --config
```

Inspect available sections:

```bash
grep -n 'List.Rules' "$(john --config)"
```

The exact configuration location depends on the John installation.

---

## 24. Using a John Rule Section

John can apply a ruleset during wordlist mode.

Example:

```bash
john \
    --format=Raw-SHA256 \
    --wordlist=words.txt \
    --rules \
    lab.hash
```

Use the installed configuration and rules supported by your John build.

---

## 25. John Incremental Comparison

Run the same laboratory hash with a wordlist:

```bash
john \
    --format=Raw-SHA256 \
    --wordlist=words.txt \
    --rules \
    lab.hash
```

Then compare it with:

```bash
john \
    --format=Raw-SHA256 \
    --incremental \
    lab.hash
```

Record:

```text
Attack
Candidates
Speed
Runtime
Result
```

---

# Practical Rule Research

## 26. Word + Number

Create:

```bash
cat > number.rule <<'EOF'
$0
$1
$2
$3
$4
$5
$6
$7
$8
$9
EOF
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r number.rule \
    --stdout
```

This generates candidates with one digit appended.

---

## 27. Word + Year

For a controlled research experiment:

```bash
cat > year.rule <<'EOF'
$2$0$2$4
$2$0$2$5
$2$0$2$6
EOF
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r year.rule \
    --stdout
```

This produces candidates such as:

```text
password2024
password2025
password2026
```

---

## 28. Capitalized Word + Year

Create:

```bash
cat > cap-year.rule <<'EOF'
c$2$0$2$4
c$2$0$2$5
c$2$0$2$6
EOF
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r cap-year.rule \
    --stdout
```

---

## 29. Word + Special Character

Create:

```bash
cat > special.rule <<'EOF'
$!
$@
$#
$$
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r special.rule \
    --stdout
```

This demonstrates predictable suffix transformations.

---

## 30. Prefix + Word

Create:

```bash
cat > prefix.rule <<'EOF'
^1
^2
^3
EOF
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r prefix.rule \
    --stdout
```

---

## 31. Case + Suffix Combination

Create:

```bash
cat > mixed.rule <<'EOF'
c$1
c$2
c$3
c$!
u$1
u$2
u$3
EOF
```

Run:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r mixed.rule \
    --stdout
```

---

# Rule Efficiency

## 32. Baseline

First run the plain wordlist:

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    words.txt
```

Record:

```text
Wordlist size
Candidates
Speed
Runtime
Result
```

---

## 33. Add Rules

Now run:

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    words.txt \
    -r custom.rule
```

Compare:

```text
Plain wordlist
vs
Wordlist + rules
```

Record:

```text
Candidate expansion
Speed
Runtime
Recovery rate
```

---

## 34. Rule Expansion

If:

```text
words.txt
```

contains:

```text
1,000 words
```

and the rule file contains:

```text
20 rules
```

the theoretical candidate count can approach:

```text
1,000 × 20
```

before duplicate removal and other effects.

Measure the actual output:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r custom.rule \
    --stdout | wc -l
```

---

## 35. Duplicate Analysis

Generate:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r custom.rule \
    --stdout > candidates.txt
```

Count all:

```bash
wc -l candidates.txt
```

Count unique:

```bash
sort -u candidates.txt | wc -l
```

The difference shows how much duplicate generation exists in the ruleset.

---

## 36. Rule Benchmark

Run:

```bash
time hashcat \
    -a 0 \
    words.txt \
    -r custom.rule \
    --stdout > /dev/null
```

Repeat with another ruleset:

```bash
time hashcat \
    -a 0 \
    words.txt \
    -r /usr/share/hashcat/rules/best64.rule \
    --stdout > /dev/null
```

Record:

```text
Rule set
Rules
Candidates
Time
```

---

## 37. Rule + Wordlist Comparison

Test:

```text
A → wordlist only
B → wordlist + small custom rules
C → wordlist + best64
D → larger ruleset
```

Create a table:

| Test | Wordlist  | Rules  | Candidates | Runtime | Result |
| ---- | --------- | ------ | ---------: | ------: | ------ |
| A    | words.txt | None   |     Record |  Record | Record |
| B    | words.txt | custom |     Record |  Record | Record |
| C    | words.txt | best64 |     Record |  Record | Record |
| D    | words.txt | larger |     Record |  Record | Record |

---

# Advanced Research

## 38. Rules and Masks

Rules and masks solve different problems.

Rules:

```text
Dictionary
    ↓
Transform
    ↓
Candidate
```

Masks:

```text
Pattern
    ↓
Generate
    ↓
Candidate
```

A structured research workflow can test both approaches against the same laboratory target.

---

## 39. Hybrid Research

Hashcat supports hybrid attack modes.

Check:

```bash
hashcat --help
```

Relevant modes include:

```text
-a 6
-a 7
```

These combine dictionary candidates and mask-generated candidates.

Use them only against your disposable laboratory hashes.

---

## 40. Dictionary + Mask

Example structure:

```text
word + two digits
```

A hybrid attack can generate:

```text
password00
password01
password02
...
```

Hashcat syntax:

```bash
hashcat \
    -m 1400 \
    -a 6 \
    lab.hash \
    words.txt \
    '?d?d'
```

---

## 41. Mask + Dictionary

Reverse the order:

```bash
hashcat \
    -m 1400 \
    -a 7 \
    lab.hash \
    '?d?d' \
    words.txt
```

This is useful for testing:

```text
two digits + word
```

Compare both hybrid directions experimentally.

---

## 42. Custom Rule Research

Create a dedicated rules file:

```bash
mkdir -p rules
nano rules/lab.rule
```

Example:

```text
:
c
$1
$2
$3
$!
c$1
c$2
c$3
c$!
```

Run:

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    words.txt \
    -r rules/lab.rule
```

Keep custom rules version-controlled.

---

## 43. Research Versioning

Record:

```text
Rule file:
Git commit:
Wordlist:
Hash mode:
Hashcat version:
Hardware:
Date:
```

When modifying the rules, create a new experiment rather than overwriting previous results.

---

## 44. Practical Experiment

Create:

```text
Redteam2026!
```

Generate:

```bash
printf '%s' 'Redteam2026!' | sha256sum
```

Save:

```bash
echo '<sha256-hash>' > lab.hash
```

Create:

```bash
cat > words.txt <<'EOF'
password
redteam
security
admin
EOF
```

Create:

```bash
cat > lab.rule <<'EOF'
:
c$1
c$2
c$3
c$!
c$2$0$2$6$!
EOF
```

Inspect:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r lab.rule \
    --stdout
```

Then run the recovery:

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    words.txt \
    -r lab.rule
```

Show the result:

```bash
hashcat \
    -m 1400 \
    lab.hash \
    --show
```

---

## 45. Advanced Experiment

Create multiple disposable passwords:

```text
redteam1
Redteam1
Redteam2026
Redteam2026!
security123
Security123!
```

Use the same base wordlist.

Test:

```text
Experiment 1 → plain dictionary
Experiment 2 → capitalization
Experiment 3 → numeric suffixes
Experiment 4 → year suffixes
Experiment 5 → special-character suffix
Experiment 6 → combined rules
```

Record which transformations recover each laboratory password.

---

## 46. Research Results

Use:

| Target | Base Word | Rule      | Recovered | Candidates | Runtime |
| ------ | --------- | --------- | --------- | ---------: | ------: |
| LAB-01 | redteam   | `:`       | No/Yes    |     Record |  Record |
| LAB-02 | redteam   | `c`       | No/Yes    |     Record |  Record |
| LAB-03 | redteam   | `c$1`     | No/Yes    |     Record |  Record |
| LAB-04 | redteam   | year rule | No/Yes    |     Record |  Record |
| LAB-05 | redteam   | combined  | No/Yes    |     Record |  Record |

Use measured values instead of estimates.

---

## 47. Troubleshooting

### Rule Produces Nothing

Check:

```bash
cat lab.rule
```

Then:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r lab.rule \
    --stdout
```

If the generated output is unexpected, inspect the rule syntax.

---

### Password Is Not Recovered

Check:

```text
Base word present?
Correct capitalization?
Correct suffix?
Correct special character?
Correct hash mode?
Correct hash?
```

A rule attack only generates the transformations defined by the rules.

---

### Too Many Candidates

Reduce the rule set.

Start with:

```text
small wordlist
+
small rule file
```

Then expand only when the experiment requires it.

---

### Duplicate Candidates

Measure:

```bash
hashcat \
    -a 0 \
    words.txt \
    -r lab.rule \
    --stdout > candidates.txt

wc -l candidates.txt
sort -u candidates.txt | wc -l
```

Optimize rules that produce unnecessary duplicates.

---

## 48. Quick Reference

### Plain Wordlist

```bash
hashcat -m 1400 -a 0 lab.hash words.txt
```

### Best64

```bash
hashcat \
    -m 1400 \
    -a 0 \
    lab.hash \
    words.txt \
    -r /usr/share/hashcat/rules/best64.rule
```

### Inspect Rules

```bash
hashcat \
    -a 0 \
    words.txt \
    -r lab.rule \
    --stdout
```

### Save Candidates

```bash
hashcat \
    -a 0 \
    words.txt \
    -r lab.rule \
    --stdout > candidates.txt
```

### Count Candidates

```bash
wc -l candidates.txt
```

### Count Unique Candidates

```bash
sort -u candidates.txt | wc -l
```

### Hybrid Word + Mask

```bash
hashcat -m 1400 -a 6 lab.hash words.txt '?d?d'
```

### Hybrid Mask + Word

```bash
hashcat -m 1400 -a 7 lab.hash '?d?d' words.txt
```

### Show Recovered Password

```bash
hashcat -m 1400 lab.hash --show
```

---

## 49. Research Checklist

```text
[ ] Authorized laboratory target
[ ] Disposable password
[ ] Hash format verified
[ ] Base wordlist documented
[ ] Rule file documented
[ ] Rule syntax tested with --stdout
[ ] Candidate count measured
[ ] Duplicate count measured
[ ] Hardware recorded
[ ] Hashcat version recorded
[ ] Candidate rate recorded
[ ] Runtime recorded
[ ] Recovery verified
[ ] Results preserved
[ ] No real credentials included
```

---

## 50. Findings

A useful rule-based attack experiment should show:

```text
Base wordlist
+
Transformation rules
=
Expanded candidate space
```

The important measurements are:

```text
Original wordlist size
Generated candidate count
Unique candidate count
Candidate rate
Runtime
Recovered candidates
Rule efficiency
```

The goal is to understand how predictable password modifications affect offline recovery cost and how efficiently a rule set explores those patterns.
