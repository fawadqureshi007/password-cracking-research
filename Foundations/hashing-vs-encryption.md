# Hashing vs Encryption

Hashing and encryption are two different cryptographic mechanisms. They are often grouped together when discussing password security, but they solve different problems and require different approaches during a security assessment.

## Hashing

A cryptographic hash function takes input data and produces a fixed-length output.

```text
Input
  ↓
Hash Function
  ↓
Hash
```

For example, a password can be processed into a hash that is stored by an application instead of storing the original password.

During authentication, the supplied password is processed again and the resulting value is compared with the stored value.

```text
Password
   ↓
Hash Function
   ↓
Hash
   ↓
Compare
   ↓
Stored Hash
```

A cryptographic hash is designed to be computationally infeasible to reverse. There is no normal "decrypt" operation for a password hash.

Password cracking against a hash therefore means testing candidate passwords until one produces the required hash.

```text
Candidate
   ↓
Hash
   ↓
Compare with target
   ↓
Match / No Match
```

The practicality of this process depends heavily on the hashing algorithm, password, available candidate space, and hardware.

---

## Encryption

Encryption is designed to transform readable data into ciphertext that can later be recovered with the appropriate key.

```text
Plaintext
   ↓
Encryption + Key
   ↓
Ciphertext
```

The intended operation is reversible when the correct key is available:

```text
Ciphertext
   ↓
Decryption + Key
   ↓
Plaintext
```

Examples include encrypted archives, protected documents, encrypted private keys, and encrypted databases.

A password may be used directly as a key or processed through a key-derivation mechanism before being used to encrypt or decrypt the data.

---

## Main Difference

| Property             | Hashing                               | Encryption                                |
| -------------------- | ------------------------------------- | ----------------------------------------- |
| Purpose              | One-way representation / verification | Confidentiality                           |
| Reversible           | No                                    | Yes, with the correct key                 |
| Output               | Hash / digest                         | Ciphertext                                |
| Typical password use | Password verification                 | Protecting data                           |
| Common assessment    | Password recovery from hashes         | Password/key recovery from protected data |
| Examples             | NTLM, bcrypt, Argon2id                | Encrypted ZIP, PDF, 7z, SSH key           |

---

## Password Hashing

Not every hash function is suitable for storing passwords.

General-purpose algorithms such as MD5 and SHA-1 are designed to be fast. That makes them unsuitable for modern password storage because an attacker can test candidates at high speed when hashes are obtained.

Modern password storage normally uses dedicated password hashing or key-derivation algorithms such as:

* Argon2id
* bcrypt
* scrypt
* PBKDF2

These algorithms deliberately increase the cost of testing each password candidate.

---

## Salting

Password hashes should normally include a unique salt.

Conceptually:

```text
Password + Unique Salt
          ↓
     Password KDF
          ↓
      Stored Result
```

The salt does not make a weak password strong. Its purpose is to prevent identical passwords from producing identical stored values and to make precomputed lookup attacks less useful.

Salts are covered separately in [Salts](salts.md).

---

## Password Cracking: Hashes

When a tester has an authorized password hash, the original password is not simply extracted from it.

Instead, candidate passwords are generated and tested.

```text
Target Hash
     │
     ▼
Candidate Generator
     │
     ▼
Candidate Password
     │
     ▼
Hash / KDF
     │
     ▼
Compare
   ┌─┴─┐
 Match  No Match
   │
   ▼
Recovered Candidate
```

The candidate-generation strategy can include:

* Dictionary attacks
* Rules
* Masks
* Brute force
* Hybrid attacks
* Custom wordlists

The appropriate method depends on what is known about the password.

---

## Password Recovery: Encrypted Files

Encrypted files work differently.

Consider a password-protected ZIP archive:

```text
Protected ZIP
     │
     ▼
Candidate Password
     │
     ▼
Password Verification / Key Derivation
     │
     ▼
Does the candidate unlock the archive?
```

The process is still based on candidate testing, but the underlying protection mechanism is different from a conventional password hash.

The same principle applies to other protected artifacts, although the implementation and supported tools vary between formats.

Examples include:

* PDF
* ZIP
* RAR
* 7z
* Microsoft Office files
* SSH private keys
* KeePass databases

---

## Why the Distinction Matters

The first step in a password-recovery assessment should be identifying the target and its protection mechanism.

For example:

```text
NTLM Hash
    → Offline hash recovery

bcrypt Hash
    → Offline password recovery using a password KDF

Encrypted ZIP
    → Archive password recovery

Encrypted SSH Private Key
    → Private-key passphrase recovery

SSH Login
    → Online authentication testing
```

Using the wrong model can lead to incorrect assumptions about the target and the available attack techniques.

---

## Research Workflow

For this project, the basic workflow is:

```text
Identify Target
      ↓
Identify Protection
      ↓
Determine Available Data
      ↓
Determine Attack Model
      ↓
Select Suitable Tool
      ↓
Select Attack Technique
      ↓
Run Controlled Experiment
      ↓
Record Results
      ↓
Analyze Security Impact
```

The protection mechanism should always be identified before selecting a cracking technique.

---

## Summary

Hashing and encryption are not interchangeable.

A hash is intended to be one-way and is commonly used for password verification. Password recovery against a hash involves testing candidates until a matching result is found.

Encryption is designed to be reversible with the correct key. When a password protects encrypted data, recovery research generally involves testing candidate passwords against the relevant protection mechanism.

Understanding this distinction is necessary before moving into practical work with John the Ripper, Hashcat, encrypted files, or online authentication testing.

---

**Next:** [Online vs Offline](online-vs-offline.md)
