# Cracking Time

Cracking time is an estimate of how long it may take to recover a password or test a defined password candidate space.

It is not a fixed value. Results depend on the target's protection mechanism, candidate-generation strategy, hardware, and the amount of information available about the password.

## Basic Model

For a simple exhaustive search, the average number of candidates tested before finding the correct password is approximately half of the total search space.

```text id="n4x8cp"
Average Candidates
≈
Total Candidates / 2
```

If the correct password is the final candidate tested, the full search space must be processed.

---

# Search Space

For a randomly generated password using a character set of size `N` and length `L`:

```text id="v7q2mz"
Search Space = N^L
```

For example, an 8-character password using only digits has:

```text id="f3k8yd"
10^8
=
100,000,000
```

possible combinations.

If a system can test `R` candidates per second:

```text id="x5m9qa"
Time ≈ Search Space / R
```

Average exhaustive-search time can be approximated as:

```text id="j8p4vn"
Average Time ≈ Search Space / (2 × R)
```

These calculations assume a complete and uniformly ordered search.

---

# Example

Consider a synthetic 6-digit password.

```text id="r2w7ks"
Character Set = 10
Length = 6

Search Space = 10^6
             = 1,000,000
```

If the test environment processes:

```text id="q6n3xf"
100,000 candidates/second
```

then the theoretical worst-case time is:

```text id="m8v2cz"
1,000,000 / 100,000
=
10 seconds
```

The average position of the correct password would be approximately halfway through the search:

```text id="p4x7yn"
≈ 5 seconds
```

This is a simplified mathematical model. Actual results depend on the target algorithm and implementation.

---

# Candidate Rate

The number of candidates that can be tested per second is often called the candidate rate or hash rate, depending on the target.

It can vary dramatically between algorithms.

For example:

```text id="d8k3vq"
Fast Hash
    ↓
High Candidate Rate

Password KDF
    ↓
Lower Candidate Rate
```

The reason is that password KDFs intentionally introduce computational and sometimes memory costs.

Therefore, a search-space calculation without considering the target algorithm can be misleading.

---

# Hardware

Hardware affects offline password-recovery performance.

Relevant factors include:

* CPU performance
* GPU performance
* Available memory
* Number of processing devices
* Thermal limits
* Driver and software configuration
* Parallelization

A useful benchmark should record the hardware used.

Example:

```text id="z5m8xc"
CPU:       [document]
GPU:       [document]
RAM:       [document]
Software:  [document]
Algorithm: [document]
```

Results from different systems should not be compared without considering these variables.

---

# Online vs Offline

Cracking-time calculations are particularly different between online and offline testing.

### Online

```text id="q3v9ka"
Candidate
   ↓
Network
   ↓
Authentication Service
   ↓
Response
```

The candidate rate may be limited by:

* Network latency
* Server response time
* Rate limiting
* Account lockout
* Authentication controls

### Offline

```text id="w6x2np"
Candidate
   ↓
Local Verification
   ↓
Result
```

There is no need to contact the authentication server for every candidate.

The limiting factor is usually the local implementation, hardware, and target algorithm.

---

# Dictionary Attacks

A dictionary attack does not normally search the entire theoretical password space.

Instead, it tests a defined candidate list.

For example:

```text id="k7p2dm"
Wordlist
   ↓
50,000 candidates
   ↓
Candidate Testing
```

If the correct password is present near the beginning of the list, recovery may be very fast.

If it is absent, the attack will fail regardless of how quickly the candidates are processed.

This is why candidate quality can be more important than raw candidate rate for human-created passwords.

---

# Rule-Based Attacks

Rules transform existing candidates into additional candidates.

For example:

```text id="m3x8qw"
password
   ↓
Password
password1
Password!
password123
...
```

The exact transformations depend on the rules being used.

A small dictionary can therefore produce a much larger candidate set.

Cracking-time measurements should account for the number of generated candidates rather than only the size of the original wordlist.

---

# Mask Attacks

Mask attacks are useful when part of the password structure is known.

For example:

```text id="c8v4zn"
Known structure:

Word + 4 digits
```

The candidate space can be calculated from the unknown positions instead of treating the entire password as random.

This can significantly reduce the search space when the pattern is accurate.

---

# KDF Cost

A password KDF can dramatically change the practical cost of candidate testing.

Conceptually:

```text id="u9q5pk"
Password
   ↓
KDF
   ↓
Cost per Candidate
```

Increasing computational or memory cost reduces the number of candidates that can be tested within a given period.

However, KDF strength and password strength are separate properties.

```text id="r7m3xf"
Strong KDF + Weak Password
        ≠
Strong Password
```

Both should be evaluated.

---

# Cracking-Time Estimates

A useful research report should state the assumptions behind an estimate.

For example:

```text id="e4y8qm"
Target:
Synthetic bcrypt hash

Attack:
Dictionary + rules

Hardware:
[document]

Candidate Rate:
[measured]

Candidates:
[document]

Result:
Recovered / Not recovered

Elapsed Time:
[document]
```

Avoid reporting a single number without explaining the conditions under which it was measured.

---

# Benchmarking

For meaningful comparisons, keep the test conditions consistent.

A simple benchmark methodology is:

```text id="n5q2vx"
Select Target
     ↓
Record Algorithm
     ↓
Record Parameters
     ↓
Record Hardware
     ↓
Select Candidate Set
     ↓
Run Test
     ↓
Record Candidate Rate
     ↓
Record Runtime
     ↓
Repeat
     ↓
Compare Results
```

When comparing two tools, use the same target, candidate set, and hardware whenever possible.

---

# Theoretical vs Observed Time

There is an important difference between a theoretical estimate and an actual measurement.

### Theoretical

Calculated from:

```text id="w8k3zr"
Search Space
+
Estimated Candidate Rate
```

### Observed

Measured during an actual controlled experiment:

```text id="p2v7mx"
Target
+
Tool
+
Hardware
+
Candidate Set
+
Measured Runtime
```

Observed results are generally more useful for benchmarking because they account for implementation overhead and the actual test environment.

---

# Factors That Can Reduce Recovery Time

The theoretical search space can become much smaller when the tester has useful information.

Examples:

```text id="g4x9qn"
Known Length
Known Character Set
Known Prefix
Known Suffix
Known Pattern
Known Words
Password Policy
Previous Passwords
Organizational Naming Patterns
```

This information can be incorporated into candidate generation.

The objective is not necessarily to test every possible string. A good assessment attempts to model how the password was likely created.

---

# Factors That Increase Recovery Time

Recovery becomes more difficult when:

* Passwords are long
* Passwords are randomly generated
* Passwords are unique
* Candidate space is large
* A strong password KDF is used
* KDF parameters are expensive
* Memory requirements are high
* No useful information about the password is available

The exact impact depends on the target and attack model.

---

# Reporting Results

A useful research result should contain enough information to reproduce the experiment.

Example:

```text id="s6x3pm"
Target:          Synthetic test hash
Algorithm:       [document]
Attack:          Dictionary + rules
Candidates:      [document]
Hardware:        [document]
Candidate Rate:  [document]
Elapsed Time:    [document]
Result:          Recovered / Not recovered
```

For public research, use synthetic credentials and lab-generated targets.

Do not publish credentials or password hashes obtained from unauthorized systems.

---

# Summary

Cracking time depends on more than password length.

A practical estimate must consider:

```text id="q8m4vz"
Password Search Space
        +
Candidate Generation
        +
Target Algorithm
        +
KDF Parameters
        +
Hardware
        +
Attack Strategy
        +
Available Information
```

For this reason, cracking-time figures should always be presented with their testing conditions.

A measured benchmark is more useful than an unsupported claim about how quickly a password "can be cracked."

