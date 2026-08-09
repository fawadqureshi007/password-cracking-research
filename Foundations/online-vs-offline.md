# Online vs Offline Password Attacks

Password attacks can be divided into two main categories: **online** and **offline**.

The difference is whether password candidates are tested directly against an authentication service or locally against data that has already been obtained during an authorized assessment.

## Online Attacks

An online attack interacts with the target authentication service for every password attempt.

```text id="3u9v3d"
Password Candidate
        ↓
Authentication Service
        ↓
   Valid / Invalid
```

For example, a controlled lab may contain an SSH service with a test account.

A tester submits a candidate password to the service. The server determines whether the credentials are valid and returns a response.

The process can be affected by controls implemented by the service.

### Common Controls

* Rate limiting
* Account lockout
* Multi-factor authentication
* CAPTCHA
* IP restrictions
* Login monitoring
* Intrusion detection
* Authentication throttling

These controls make online password attacks fundamentally different from offline cracking.

---

## Offline Attacks

An offline attack does not require sending every candidate to the authentication service.

Instead, the tester has obtained an authorized representation of the password or protected data and performs candidate testing locally.

For a password hash:

```text id="5q4glp"
Target Hash
     ↓
Candidate Password
     ↓
Hash / KDF
     ↓
Compare
     ↓
Match / No Match
```

For an encrypted file:

```text id="8v0qjf"
Protected File
      ↓
Candidate Password
      ↓
Protection / KDF
      ↓
Verification
      ↓
Valid / Invalid
```

Tools such as **John the Ripper** and **Hashcat** are commonly used for offline password-recovery research.

---

# Online vs Offline

| Property                        | Online                 | Offline                      |
| ------------------------------- | ---------------------- | ---------------------------- |
| Target                          | Authentication service | Hash / protected artifact    |
| Network required                | Yes                    | Usually no                   |
| Server involved in each attempt | Yes                    | No                           |
| Rate limiting                   | Possible               | Not normally                 |
| Account lockout                 | Possible               | Not applicable               |
| Local hardware                  | Less important         | Very important               |
| Candidate testing speed         | Service-dependent      | Hardware/algorithm-dependent |
| Common tools                    | Medusa, Hydra          | John, Hashcat                |

---

## Online Password Guessing

Online testing involves sending credentials to an authentication endpoint.

A simplified model:

```text id="p7b3g0"
Candidate 1 → Login → Failed
Candidate 2 → Login → Failed
Candidate 3 → Login → Failed
Candidate 4 → Login → Success
```

The target can detect these attempts because they occur through its normal authentication process.

For that reason, online testing is useful not only for evaluating password strength but also for evaluating defensive controls.

A properly configured service should make large-scale guessing difficult through mechanisms such as rate limiting, MFA, lockout policies, and monitoring.

---

## Password Spraying

Password spraying is different from repeatedly attacking one account with many passwords.

The basic idea is:

```text id="08at16"
Password A
   ↓
Multiple Test Accounts
```

Instead of:

```text id="8q2hup"
Account A
   ↓
Many Passwords
```

The objective of a controlled password-spraying assessment is to determine whether weak or reused passwords can expose multiple accounts while avoiding unnecessary account lockouts.

This type of testing should only be performed against explicitly authorized accounts and infrastructure.

---

## Offline Hash Recovery

Offline recovery removes the authentication service from the candidate-testing loop.

For example:

```text id="j6x7pg"
Obtained Lab Hash
       ↓
Candidate Generator
       ↓
Hash Function / KDF
       ↓
Comparison
       ↓
Result
```

The tester can potentially evaluate a very large number of candidates without generating login events on the original service.

This makes the strength of the password-storage mechanism extremely important.

---

## Why Offline Attacks Can Be More Serious

Once an attacker obtains a password database containing suitable password representations, the authentication server may no longer be involved in the recovery process.

The attacker can work on the data independently.

The practical difficulty then depends on factors such as:

* Password strength
* Hash or KDF algorithm
* KDF parameters
* Salt configuration
* Candidate-generation strategy
* Available hardware
* Number of targets
* Amount of known information

This is one reason secure password storage is important even when an application already has login protections.

---

# Encrypted Files and Offline Recovery

Encrypted files are also commonly evaluated offline.

Examples include:

```text id="u0ah5v"
PDF
ZIP
RAR
7z
Office documents
SSH private keys
KeePass databases
```

The tester creates or obtains an authorized protected artifact and evaluates whether its password can be recovered using appropriate techniques.

The exact process depends on the format and protection scheme.

---

# Choosing the Attack Model

The target determines the attack model.

```text id="p9t3bw"
Authentication Service
        ↓
      ONLINE
        ↓
Medusa / Hydra
```

```text id="gk8k4e"
Password Hash
        ↓
      OFFLINE
        ↓
John / Hashcat
```

```text id="2lj2e9"
Encrypted File
        ↓
      OFFLINE
        ↓
Format-specific recovery workflow
```

The same password may therefore have very different practical security depending on whether an attacker can only interact with an online service or has obtained offline password-verification data.

---

# Red-Team Assessment Workflow

A controlled assessment can be structured as follows:

```text id="3qk0lw"
Identify Authentication Surface
            ↓
Determine Available Controls
            ↓
Test Rate Limiting
            ↓
Test Lockout Behavior
            ↓
Evaluate Password Policy
            ↓
Check for Credential Reuse
            ↓
If Authorized Hashes Are Available
            ↓
Perform Offline Recovery
            ↓
Compare Findings
            ↓
Report Risk and Mitigation
```

The online and offline portions should be documented separately because they measure different security properties.

---

# Key Takeaways

**Online attacks** depend on the target service and its defensive controls.

**Offline attacks** depend heavily on the quality of the password protection mechanism and the resources available to the tester.

Online testing is primarily an assessment of the authentication surface.

Offline testing is primarily an assessment of password strength and password-storage or encryption mechanisms.

Both can be relevant during an authorized red-team engagement, but they should not be treated as the same attack.

---

**Next:** [Salts](salts.md)
