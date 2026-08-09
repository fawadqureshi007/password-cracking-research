# Attack Techniques

Practical research into password-attack techniques used in authorized security testing, CTFs, and controlled laboratory environments.

This section focuses on **how different attack strategies work, when they are useful, and how to measure their effectiveness**.

---

## Techniques

| Technique                                       | Description                                          |
| ----------------------------------------------- | ---------------------------------------------------- |
| [Dictionary Attacks](dictionary-attacks.md)     | Test candidate passwords from a wordlist             |
| [Brute-Force Attacks](brute-force-attacks.md)   | Exhaustively test a defined character space          |
| [Mask Attacks](mask-attacks.md)                 | Target passwords matching a known structure          |
| [Rule-Based Attacks](rule-based-attacks.md)     | Transform base words into password candidates        |
| [Hybrid Attacks](hybrid-attacks.md)             | Combine words with generated characters              |
| [Combinator Attacks](combinator-attacks.md)     | Combine candidate words into larger candidates       |
| [Incremental Attacks](incremental-attacks.md)   | Progressively search increasingly complex candidates |
| [Distributed Cracking](distributed-cracking.md) | Split authorized offline workloads across systems    |

---

## Attack Selection

The technique should match what is known about the password.

```text
Known password words
        ↓
Dictionary Attack

Known password pattern
        ↓
Mask Attack

Known words + predictable modifications
        ↓
Rule-Based Attack

Known word + unknown suffix/prefix
        ↓
Hybrid Attack

Two or more wordlists
        ↓
Combinator Attack

No useful information
        ↓
Brute-Force / Incremental
```

---

## Practical Workflow

```text
Authorized Target
       ↓
Identify Credential Material
       ↓
Identify Hash / Encryption Format
       ↓
Estimate Search Space
       ↓
Select Attack Technique
       ↓
Build Candidate Set
       ↓
Benchmark
       ↓
Run Offline Recovery
       ↓
Record Results
       ↓
Verify Recovery
       ↓
Document Findings
```

---

## Dictionary Attacks

Dictionary attacks use an existing collection of candidate passwords.

Typical sources include:

```text
Common passwords
Password dictionaries
Organization-specific terms
Synthetic laboratory lists
Previously generated candidates
```

Useful when the password is likely to contain common or predictable words.

See:

**[dictionary-attacks.md](dictionary-attacks.md)**

---

## Brute-Force Attacks

Brute force systematically tests candidates from a defined character space.

Example concept:

```text
aaaa
aaab
aaac
...
zzzz
```

The search space increases rapidly as password length and character-set size increase.

See:

**[brute-force-attacks.md](brute-force-attacks.md)**

---

## Mask Attacks

Mask attacks are useful when part of the password structure is known.

Example:

```text
Known prefix + unknown digits
```

Conceptually:

```text
Company202?
Company201?
Company200?
...
```

Masks reduce unnecessary candidates when password structure is known.

See:

**[mask-attacks.md](mask-attacks.md)**

---

## Rule-Based Attacks

Rules transform existing words into candidate passwords.

Typical transformations include:

```text
Capitalization
Number suffixes
Character substitutions
Prefixes
Suffixes
Symbol additions
```

Example concept:

```text
password
Password
password1
Password1
Password!
Password2026
```

See:

**[rule-based-attacks.md](rule-based-attacks.md)**

---

## Hybrid Attacks

Hybrid attacks combine a wordlist with generated characters.

Example:

```text
Word + digits
Word + symbols
Digits + word
```

They are useful when part of the password is predictable and another portion is unknown.

See:

**[hybrid-attacks.md](hybrid-attacks.md)**

---

## Combinator Attacks

Combinator attacks combine entries from multiple candidate lists.

Example:

```text
red + team
redteam + 2026
security + lab
```

This is useful when passwords are constructed from multiple recognizable components.

See:

**[combinator-attacks.md](combinator-attacks.md)**

---

## Incremental Attacks

Incremental attacks progressively explore candidate spaces without relying entirely on a predefined dictionary.

They can become computationally expensive very quickly.

Use them primarily for controlled experiments and benchmarking.

See:

**[incremental-attacks.md](incremental-attacks.md)**

---

## Distributed Cracking

Large offline workloads can be divided between multiple authorized systems.

Conceptually:

```text
                 Controller
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
     Worker 1     Worker 2     Worker 3
        │            │            │
        └────────────┼────────────┘
                     ↓
                  Results
```

Research should record:

```text
Number of workers
Hardware
Candidate allocation
Hash algorithm
Processing speed
Total candidates
Runtime
```

See:

**[distributed-cracking.md](distributed-cracking.md)**

---

## Offline vs Online

These techniques are primarily relevant to **offline password recovery**.

```text
Offline
-------
Hash / encrypted credential
        ↓
Local candidate testing
        ↓
No authentication request required


Online
------
Username + password candidate
        ↓
Authentication service
        ↓
Server response
```

Online authentication testing has additional concerns such as:

```text
Rate limits
Account lockout
MFA
Detection
Authorization
Provider policies
```

Do not apply offline cracking workflows to live authentication services without explicit authorization.

---

## Measuring an Attack

Every experiment should record more than whether a password was recovered.

Recommended measurements:

```text
Attack technique
Hash / format
Candidate count
Candidate generation method
Hardware
Tool
Tool version
Hashing/KDF algorithm
Processing speed
Runtime
Password recovered
```

Example:

```text
Technique: Dictionary
Candidates: 100,000
Algorithm: <algorithm>
Tool: Hashcat
Hardware: <hardware>
Speed: <measured speed>
Runtime: <measured time>
Result: Recovered / Not recovered
```

---

## Choosing the Technique

Use available information before selecting the attack.

```text
Nothing known
    ↓
Dictionary
    ↓
Rules
    ↓
Mask
    ↓
Hybrid
    ↓
Combinator
    ↓
Brute Force
```

This is not a fixed order. The most efficient strategy depends on the password characteristics, candidate space, algorithm, and available hardware.

---

## Research Principle

The objective of this section is not simply:

```text
"Crack a password."
```

The objective is to measure:

```text
Why did the attack work?
Why did it fail?
How large was the candidate space?
How expensive was the computation?
Which technique was most efficient?
How did password construction affect recovery?
How did the hashing/KDF configuration affect recovery?
```

---

## Authorization

All practical testing should use:

* Your own credentials
* Synthetic laboratory data
* CTF environments
* Dedicated test systems
* Explicitly authorized security assessments

Do not use these techniques to obtain unauthorized credentials or access accounts and systems without permission.

---

## Research Status

This directory contains practical research material. Results should be reproducible wherever possible, with the hardware, software versions, attack parameters, and test data documented alongside experiments.
