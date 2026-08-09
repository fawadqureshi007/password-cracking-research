# Key Derivation Functions

A key derivation function (KDF) is a function that derives a cryptographic key or password-derived value from input material such as a password.

KDFs are important in password security because they can make password guessing more expensive.

A general-purpose hash is designed to be efficient. A password KDF is designed to make large numbers of password guesses more costly.

## Basic Model

```text id="v8u1mq"
Password
   +
Salt
   +
Cost Parameters
   ↓
   KDF
   ↓
Derived Value
```

During authentication, the supplied password is processed using the same parameters and compared with the stored result.

---

# Why KDFs Matter

Suppose an attacker obtains password-verification data.

With a very fast hashing algorithm, the attacker may be able to test a large number of candidates quickly.

With a password KDF:

```text id="m9r3qa"
Candidate Password
       ↓
    KDF Cost
       ↓
Candidate Result
       ↓
     Compare
```

Each candidate requires additional computation and, depending on the algorithm, memory.

The objective is not to make password verification impossible. It is to make large-scale guessing significantly more expensive.

---

# Common Password KDFs

Several algorithms are widely used for password protection.

## Argon2

Argon2 was designed specifically for password hashing and key derivation.

Argon2id is commonly recommended for password storage because it combines resistance characteristics from Argon2's different modes.

Important parameters include:

* Memory cost
* Time cost
* Parallelism
* Salt

Conceptually:

```text id="x8j3pd"
Password
   +
Salt
   +
Memory Cost
   +
Time Cost
   +
Parallelism
   ↓
 Argon2id
   ↓
Derived Value
```

---

## bcrypt

bcrypt is an established password-hashing algorithm based on the Blowfish key schedule.

Its primary configurable parameter is the cost factor.

Increasing the cost increases the computational work required for password verification.

bcrypt also incorporates a salt into its password-processing scheme.

---

## scrypt

scrypt was designed to make password guessing expensive in both computation and memory.

Its parameters include:

* CPU/memory cost
* Block size
* Parallelization
* Salt

The memory requirement is particularly relevant when an attacker attempts large-scale parallel password recovery.

---

## PBKDF2

PBKDF2 derives a value by repeatedly applying a pseudorandom function.

Its primary cost parameter is the iteration count.

Conceptually:

```text id="s7q4ny"
Password + Salt
      ↓
Repeated Derivation
      ↓
Derived Value
```

A higher iteration count increases the work required for each candidate.

The appropriate configuration depends on the selected pseudorandom function and current security requirements.

---

# KDF Parameters

The algorithm alone does not determine the cost of password recovery.

Parameters matter.

For example:

```text id="q3w9ka"
Algorithm
   +
Salt
   +
Memory
   +
Iterations / Time Cost
   +
Parallelism
   ↓
Effective Password-Testing Cost
```

Two systems using the same KDF can have very different security characteristics if their parameters differ significantly.

---

# Fast Hash vs Password KDF

Consider two simplified systems.

### Fast Hash

```text id="t1z5cv"
Password
   ↓
Fast Hash
   ↓
Stored Hash
```

The algorithm is optimized for speed.

That is useful for many cryptographic applications but undesirable for password storage.

### Password KDF

```text id="r7m2qx"
Password
   +
Salt
   ↓
Expensive KDF
   ↓
Stored Derived Value
```

The additional work is intentional.

An attacker attempting millions of password candidates has to pay the configured cost for every candidate.

---

# KDFs and Offline Password Recovery

KDF configuration becomes particularly important when password-verification data is available offline.

The general process is:

```text id="v6q2hm"
Candidate Password
       ↓
Salt + KDF Parameters
       ↓
KDF
       ↓
Candidate Result
       ↓
Compare With Target
```

The attacker can repeat this process using different candidates.

A stronger KDF does not make a weak password impossible to recover. It increases the cost of each candidate test.

This distinction is important when interpreting cracking results.

---

# Memory-Hard Functions

Some password KDFs deliberately require substantial memory.

This changes the economics of large-scale password recovery.

A simplified comparison:

```text id="h8v3yx"
CPU-focused cost
      vs
CPU + Memory cost
```

Memory requirements can limit how many candidates can be processed in parallel.

scrypt and Argon2 are examples of password KDFs designed with memory-hard properties.

---

# Choosing Parameters

KDF parameters should be selected according to the environment in which passwords are verified.

A server can generally tolerate more work per login than a system handling extremely large authentication volumes.

The configuration should therefore consider:

* Available server resources
* Authentication frequency
* Expected user population
* Hardware
* Security requirements
* Current recommendations

The goal is to make password verification sufficiently expensive without creating an impractical user experience or service load.

---

# KDF Research

During an authorized assessment, useful information to document includes:

```text id="e4t8zr"
Algorithm
Version
Salt
Salt Length
Memory Cost
Time Cost
Iteration Count
Parallelism
Derived-Key Length
```

A research record might look like:

```text id="j8s3mx"
Algorithm:       [document]
Salt:            Unique
Memory Cost:     [document]
Time Cost:       [document]
Iterations:      [document]
Parallelism:     [document]
```

Do not publish real credentials or sensitive password databases in research repositories.

---

# Comparing KDF Configurations

A useful experiment is to create several synthetic password records using different configurations.

For example:

```text id="q0p6wd"
Configuration A
    ↓
Low computational cost

Configuration B
    ↓
Higher computational cost

Configuration C
    ↓
Higher memory requirement
```

The experiment can then measure:

* Verification time
* Candidate-processing rate
* CPU utilization
* Memory utilization
* Scaling behavior

The same test methodology should be used for each configuration.

---

# KDFs Do Not Replace Good Passwords

A strong KDF is one layer of protection.

It does not compensate indefinitely for a weak password.

For example:

```text id="b2v7qa"
Weak Password
      +
Strong KDF
      =
Better protection than a fast hash,
but still a weak password
```

A secure password-storage design should combine an appropriate KDF, unique salts, suitable parameters, and strong password practices.

---

# Research Workflow

When evaluating a password KDF:

```text id="k4x8fz"
Identify KDF
     ↓
Identify Parameters
     ↓
Check Salt Handling
     ↓
Create Controlled Test Data
     ↓
Measure Verification Cost
     ↓
Evaluate Candidate Recovery
     ↓
Compare With Recommended Configuration
     ↓
Document Findings
```

This provides more useful information than simply reporting whether a password was recovered.

---

# Summary

Password KDFs are designed to increase the cost of password guessing.

Important examples include:

* Argon2id
* bcrypt
* scrypt
* PBKDF2

Their security depends on both the algorithm and its configuration.

For password-cracking research, the KDF determines a significant part of the cost of testing each candidate. For defenders, selecting an appropriate KDF and configuration makes offline password recovery substantially more difficult.

