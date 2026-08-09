# Incremental Attacks

Practical guide to John the Ripper incremental password searching.

Incremental mode systematically generates candidate passwords from a configured character set. Unlike a targeted dictionary attack, it does not require a wordlist.

Use this only against hashes you own or are explicitly authorized to test.

---

## 1. Verify John

```bash
john --version
```

Check available formats:

```bash
john --list=formats
```

Check help:

```bash
john --help
```

---

# Lab Setup

## 2. Create a Disposable Password

Use a short laboratory password so the experiment finishes quickly.

Example:

```text
a7
```

Generate a SHA-256 hash:

```bash
printf '%s' 'a7' | sha256sum
```

Save the resulting hash:

```bash
echo '<sha256-hash>' > lab.hash
```

---

## 3. Verify the Hash

```bash
cat lab.hash
```

Check the format:

```bash
john --format=raw-sha256 lab.hash
```

If John reports that the hash format is not accepted, check:

```bash
john --list=formats | grep -i sha
```

---

# Incremental Mode

## 4. Basic Command

John's incremental mode is selected with:

```bash
--incremental
```

Basic syntax:

```bash
john --incremental lab.hash
```

John begins generating candidates according to its incremental configuration.

---

## 5. Specify the Format

For a raw SHA-256 laboratory hash:

```bash
john \
    --format=raw-sha256 \
    --incremental \
    lab.hash
```

---

## 6. Show Recovered Password

```bash
john --show lab.hash
```

With the format:

```bash
john \
    --format=raw-sha256 \
    --show \
    lab.hash
```

---

# Incremental Configurations

## 7. List Configuration

John's configuration is normally stored in:

```text
john.conf
```

Common locations include:

```text
/etc/john/john.conf
```

or:

```text
/usr/share/john/john.conf
```

Find it:

```bash
locate john.conf 2>/dev/null
```

Or:

```bash
find /etc /usr/share -name john.conf 2>/dev/null
```

---

## 8. Inspect Incremental Sections

Search:

```bash
grep -n '\[Incremental:' /etc/john/john.conf 2>/dev/null
```

If that location does not exist:

```bash
grep -n '\[Incremental:' /usr/share/john/john.conf 2>/dev/null
```

You may see sections such as:

```text
[Incremental:ASCII]
[Incremental:LM_ASCII]
[Incremental:Alnum]
```

The exact configurations depend on the John build.

---

# Using a Named Incremental Mode

## 9. ASCII

If your configuration contains an `ASCII` incremental section:

```bash
john \
    --format=raw-sha256 \
    --incremental=ASCII \
    lab.hash
```

---

## 10. Alphanumeric

If an `Alnum` section exists:

```bash
john \
    --format=raw-sha256 \
    --incremental=Alnum \
    lab.hash
```

The exact candidate character set comes from your John configuration.

---

# Password Length

## 11. Why Length Matters

A brute-force search grows rapidly as password length increases.

For a character set of size:

```text
C
```

and password length:

```text
L
```

the theoretical space for exactly that length is:

```text
C^L
```

For lengths from 1 through L:

```text
C^1 + C^2 + ... + C^L
```

Incremental mode determines the order in which candidates are tested based on its configured statistical model.

---

# Short Laboratory Test

## 12. Use a Tiny Password

Create:

```text
a7
```

Hash:

```bash
printf '%s' 'a7' | sha256sum
```

Save:

```bash
echo '<sha256-hash>' > lab.hash
```

Run:

```bash
john \
    --format=raw-sha256 \
    --incremental=Alnum \
    lab.hash
```

Check:

```bash
john --show lab.hash
```

---

# Sessions

## 13. Stop an Attack

Press:

```text
Ctrl+C
```

John saves its recovery state.

---

## 14. Resume

```bash
john --restore
```

If multiple sessions exist:

```bash
john --status
```

Then restore the appropriate session.

---

## 15. List Sessions

```bash
john --status
```

This can show information about active or saved work depending on the John build.

---

# Time-Limited Experiments

## 16. Run for a Fixed Period

Use the operating system's `timeout` command:

```bash
timeout 60s john \
    --format=raw-sha256 \
    --incremental=Alnum \
    lab.hash
```

This gives you a controlled one-minute experiment.

---

## 17. Run for Five Minutes

```bash
timeout 300s john \
    --format=raw-sha256 \
    --incremental=Alnum \
    lab.hash
```

Then:

```bash
john --show lab.hash
```

---

# Benchmarking

## 18. Run John Benchmark

```bash
john --test
```

For a specific format:

```bash
john \
    --test \
    --format=raw-sha256
```

Record:

```text
John version
Hash format
Hardware
Candidate rate
Benchmark result
```

---

## 19. Compare Formats

Check supported formats:

```bash
john --list=formats
```

Benchmark the relevant format:

```bash
john \
    --test \
    --format=raw-sha256
```

Do not compare speeds between different hash algorithms as though they represent the same password-recovery workload.

---

# Candidate Rate

## 20. Monitor Progress

During an active John session, press:

```text
s
```

Depending on the build, John displays status information such as:

```text
guesses
candidate rate
elapsed time
progress
```

Record the values periodically.

---

# Incremental Research

## 21. Experiment 1 — Short Password

Target:

```text
a7
```

Run:

```bash
john \
    --format=raw-sha256 \
    --incremental=Alnum \
    lab.hash
```

Record:

```text
Start time
End time
Password length
Candidate rate
Recovery status
```

---

## 22. Experiment 2 — Different Length

Create another disposable target.

Example:

```text
ab7
```

Hash:

```bash
printf '%s' 'ab7' | sha256sum
```

Replace the laboratory hash and run:

```bash
john \
    --format=raw-sha256 \
    --incremental=Alnum \
    lab.hash
```

Compare with the previous experiment.

---

## 23. Experiment 3 — Longer Target

Use a controlled laboratory password such as:

```text
abc7
```

Generate:

```bash
printf '%s' 'abc7' | sha256sum
```

Run the same configuration.

Record:

```text
Password length
Runtime
Candidate rate
Recovery result
```

Do not use real credentials for these experiments.

---

# Custom Incremental Configuration

## 24. Locate Configuration

```bash
find /etc /usr/share -name john.conf 2>/dev/null
```

Open the relevant configuration:

```bash
less /path/to/john.conf
```

Find:

```text
[Incremental:
```

Example:

```bash
grep -n '\[Incremental:' /path/to/john.conf
```

---

## 25. Inspect Character Sets

Search for charset references:

```bash
grep -n 'File =' /path/to/john.conf
```

John builds may use charset files associated with incremental modes.

Locate them:

```bash
find /usr/share -iname '*.chr' 2>/dev/null
```

Common locations may include:

```text
/usr/share/john/
```

or:

```text
/usr/share/john/charsets/
```

The exact path depends on the package and installation.

---

# Charset Training

## 26. Inspect Existing Charset Files

```bash
find /usr/share/john -name '*.chr' -print
```

Do not modify the installed files directly.

Copy a file before experimenting:

```bash
cp /path/to/example.chr ./lab.chr
```

---

## 27. Generate a Charset

John can build a charset from a training file using:

```bash
john --make-charset=lab.chr training.txt
```

Create a disposable training file:

```bash
cat > training.txt <<'EOF'
password
admin
security
redteam
cyber
laboratory
EOF
```

Generate:

```bash
john \
    --make-charset=lab.chr \
    training.txt
```

The resulting charset is for controlled research.

---

# Using a Custom Charset

## 28. Test Custom Charset

After generating a charset:

```bash
john \
    --incremental=lab \
    lab.hash
```

The exact incremental section name and charset loading behavior depend on your John configuration.

Check your configuration before using a custom setup.

---

# Character-Space Experiments

## 29. Lowercase Model

A lowercase alphabet contains:

```text
26 characters
```

For exactly four characters:

```text
26^4
```

For exactly five:

```text
26^5
```

Calculate with Python or a calculator rather than manually estimating large spaces.

---

## 30. Alphanumeric Model

A basic alphanumeric set contains:

```text
26 lowercase
26 uppercase
10 digits
```

Total:

```text
62 characters
```

For exactly four characters:

```text
62^4
```

For exactly eight:

```text
62^8
```

Theoretical keyspace increases exponentially with length.

---

# Practical Measurements

## 31. Record Runtime

Use:

```bash
/usr/bin/time -v john \
    --format=raw-sha256 \
    --incremental=Alnum \
    lab.hash
```

Useful measurements include:

```text
Elapsed time
CPU time
Memory
```

---

## 32. Fixed-Time Benchmark

```bash
timeout 60s john \
    --format=raw-sha256 \
    --incremental=Alnum \
    lab.hash
```

Then:

```bash
john --show lab.hash
```

Repeat the experiment several times for more reliable measurements.

---

# Comparing Attack Methods

## 33. Incremental vs Dictionary

Use the same laboratory hash.

Dictionary:

```bash
john \
    --format=raw-sha256 \
    --wordlist=words.txt \
    lab.hash
```

Incremental:

```bash
john \
    --format=raw-sha256 \
    --incremental=Alnum \
    lab.hash
```

Record:

| Method      | Candidate Source | Runtime | Result |
| ----------- | ---------------- | ------: | ------ |
| Dictionary  | `words.txt`      |  Record | Record |
| Incremental | `Alnum`          |  Record | Record |

---

# Incremental vs Mask

## 34. Compare With Hashcat

For a controlled experiment, Hashcat can test a known mask:

```bash
hashcat \
    -m 1400 \
    -a 3 \
    lab.hash \
    '?l?l?d'
```

John:

```bash
john \
    --format=raw-sha256 \
    --incremental=Alnum \
    lab.hash
```

The two approaches are not equivalent.

A mask specifies a precise structure.

Incremental mode uses John's incremental candidate-generation model.

---

# Resource Monitoring

## 35. CPU Usage

Linux:

```bash
top
```

or:

```bash
htop
```

Monitor John while it is running.

---

## 36. Process Information

```bash
ps aux | grep '[j]ohn'
```

---

## 37. System Load

```bash
uptime
```

Record system load when benchmarking so experiments can be compared fairly.

---

# Research Workflow

## 38. Controlled Workflow

```text
Create disposable password
        ↓
Generate hash
        ↓
Identify hash format
        ↓
Select incremental configuration
        ↓
Run controlled test
        ↓
Measure candidate rate
        ↓
Measure runtime
        ↓
Check recovery
        ↓
Record results
```

---

# Experiment Dataset

## 39. Recommended Structure

```text
incremental-research/
├── hashes/
│   ├── lab-01.hash
│   ├── lab-02.hash
│   └── lab-03.hash
├── training/
│   └── training.txt
├── charsets/
│   └── lab.chr
├── results/
│   ├── experiment-01.txt
│   ├── experiment-02.txt
│   └── experiment-03.txt
└── notes/
    └── observations.md
```

Keep real credentials and unauthorized hashes outside the project.

---

# Advanced Lab

## 40. Three Controlled Targets

Create:

```text
a7
ab7
abc7
```

Generate hashes:

```bash
printf '%s' 'a7' | sha256sum
printf '%s' 'ab7' | sha256sum
printf '%s' 'abc7' | sha256sum
```

Save them separately:

```text
lab-01.hash
lab-02.hash
lab-03.hash
```

Run the same configuration against each.

Record:

```text
Target
Length
Runtime
Candidate rate
Recovered
```

---

## 41. Compare Results

| Target | Length | Configuration | Runtime | Recovered |
| ------ | -----: | ------------- | ------: | --------- |
| `a7`   |      2 | Alnum         |  Record | Yes/No    |
| `ab7`  |      3 | Alnum         |  Record | Yes/No    |
| `abc7` |      4 | Alnum         |  Record | Yes/No    |

The purpose is to observe how increasing password length changes the search workload.

---

# Advanced Session Test

## 42. Start

```bash
john \
    --format=raw-sha256 \
    --incremental=Alnum \
    lab.hash
```

Interrupt:

```text
Ctrl+C
```

Restore:

```bash
john --restore
```

Check:

```bash
john --show lab.hash
```

This verifies that the recovery state survives interruption.

---

# Troubleshooting

## 43. Format Error

Check:

```bash
john --list=formats | grep -i sha
```

Then specify the appropriate format:

```bash
john \
    --format=<format> \
    --incremental \
    lab.hash
```

---

## 44. Incremental Mode Not Found

Check:

```bash
grep -n '\[Incremental:' /path/to/john.conf
```

If the desired mode does not exist, inspect the configuration shipped with your John build.

---

## 45. No Password Recovered

Check:

```text
Hash value
Hash format
Incremental configuration
Character set
Password length
John version
```

Verify the target independently:

```bash
printf '%s' 'LAB-PASSWORD' | sha256sum
```

Compare it with:

```bash
cat lab.hash
```

---

## 46. Search Is Too Large

Stop the experiment:

```text
Ctrl+C
```

Then reduce the research scope:

```text
Use a shorter laboratory password
Use a smaller character set
Use a targeted dictionary
Use a known mask
Limit the experiment duration
```

Do not let an uncontrolled experiment run indefinitely.

---

# Quick Reference

## Basic Incremental

```bash
john \
    --incremental \
    lab.hash
```

## Specific Format

```bash
john \
    --format=raw-sha256 \
    --incremental \
    lab.hash
```

## Named Mode

```bash
john \
    --format=raw-sha256 \
    --incremental=Alnum \
    lab.hash
```

## Show Result

```bash
john --show lab.hash
```

## Restore

```bash
john --restore
```

## Status

```bash
john --status
```

## Benchmark

```bash
john --test
```

## Format Benchmark

```bash
john \
    --test \
    --format=raw-sha256
```

## Locate Configuration

```bash
find /etc /usr/share -name john.conf 2>/dev/null
```

## Find Incremental Sections

```bash
grep -n '\[Incremental:' /path/to/john.conf
```

## Find Charset Files

```bash
find /usr/share/john -name '*.chr' -print
```

## Generate Charset

```bash
john \
    --make-charset=lab.chr \
    training.txt
```

## Fixed-Time Experiment

```bash
timeout 60s john \
    --format=raw-sha256 \
    --incremental=Alnum \
    lab.hash
```

---

# Research Checklist

```text
[ ] Disposable laboratory password
[ ] Authorized hash
[ ] Hash format verified
[ ] John version recorded
[ ] Incremental configuration recorded
[ ] Character set documented
[ ] Password length documented
[ ] Candidate rate recorded
[ ] Runtime recorded
[ ] CPU/hardware recorded
[ ] Session restoration tested
[ ] Recovery verified
[ ] Results preserved
[ ] No real credentials included
```

---

# Findings

Incremental attacks are useful for measuring how password-search workload changes as the candidate space grows.

For every experiment, record:

```text
Hash format
Incremental configuration
Character set
Password length
Candidate rate
Runtime
Hardware
Recovery result
```

Keep experiments small and reproducible. Use targeted attacks when the password structure is known rather than blindly expanding the search space.
