# Salts

A salt is a unique, random value added to a password before the password is processed by a password-hashing or key-derivation function.

The main purpose of a salt is to ensure that the same password does not produce the same stored result across different password records.

## Basic Concept

Without a salt:

```text id="r8v9k2"
Password
   ↓
Hash / KDF
   ↓
Hash
```

If two users choose the same password, they can end up with the same stored hash.

With a unique salt:

```text id="m1z2qa"
Password + Salt A
       ↓
     KDF
       ↓
    Result A


Password + Salt B
       ↓
     KDF
       ↓
    Result B
```

Even when the passwords are identical, the resulting values are different.

---

## Why Salts Matter

Consider two accounts using the same password:

```text id="x8n3qp"
User A → "password123"
User B → "password123"
```

Without unique salts:

```text id="n8x3la"
User A → Hash X
User B → Hash X
```

With unique salts:

```text id="m4j7yd"
User A → Salt A + Hash A
User B → Salt B + Hash B
```

The attacker can no longer assume that identical stored values represent identical passwords.

---

## Salts and Precomputed Attacks

One of the important purposes of salting is reducing the usefulness of precomputed hash databases and rainbow tables.

Without salts, an attacker can prepare hashes for large numbers of commonly used passwords in advance.

Conceptually:

```text id="y5r8sw"
Common Passwords
      ↓
Precomputed Hashes
      ↓
Lookup Target Hash
```

With a unique salt:

```text id="v2q6ks"
Password + Unique Salt
          ↓
       KDF / Hash
          ↓
    Target Result
```

The attacker must account for the specific salt when testing candidates.

A precomputed table created for one salt is not directly reusable for another salt.

---

## Salt Does Not Make a Weak Password Strong

A salt is not a replacement for a strong password or a suitable password KDF.

For example:

```text id="f9q1hc"
Weak Password
      +
Unique Salt
      +
Fast Hash
      =
Still Weak
```

A salt mainly changes how the password is represented and prevents certain attacks from being reused efficiently across records.

The underlying password still matters.

---

## Salt Requirements

For password storage, salts should generally be:

* Unique for each password record
* Generated using a secure random source
* Stored alongside the password hash or KDF output
* Long enough for the chosen implementation
* Generated independently rather than derived from the password

The salt does not need to be secret.

A typical stored password record may conceptually look like:

```text id="x0bq1v"
Algorithm
Cost Parameters
Salt
Derived Password Value
```

The exact format depends on the password-storage system.

---

# Salt vs Password

A salt and password serve different purposes.

| Property        | Password              | Salt                                 |
| --------------- | --------------------- | ------------------------------------ |
| Secret          | Yes                   | No                                   |
| Chosen by user  | Usually               | No                                   |
| Random          | Ideally               | Yes                                  |
| Stored publicly | No                    | Normally yes                         |
| Purpose         | Authentication secret | Make each password derivation unique |

The salt can be stored in the same database as the password hash without defeating its purpose.

---

# Salt vs Pepper

A **pepper** is different from a salt.

A salt is normally stored with the password record.

A pepper is an additional secret value kept separately from the password database.

Conceptually:

```text id="4rq2tv"
Password
   +
Salt
   +
Pepper
   ↓
Password KDF
   ↓
Stored Result
```

The security properties and operational requirements of peppers differ from salts.

A pepper should not be treated as simply another publicly stored salt.

---

# Impact on Password Cracking

When performing authorized offline password-recovery research, the salt is part of the password-verification input.

For a salted password hash:

```text id="9h3vcz"
Candidate Password
       +
Known Salt
       ↓
Password KDF
       ↓
Candidate Result
       ↓
Compare
```

The salt does not prevent password cracking if an attacker has the password record and the KDF is known.

Instead, it prevents attackers from cheaply reusing precomputed results across different salts.

The cost of testing each candidate is still primarily determined by the password KDF and its parameters.

---

# Multiple Users

Consider three accounts using the same password:

```text id="w4k0xq"
Password: "ExamplePassword123"
```

Without unique salts:

```text id="p7k6fz"
Account A → Same Result
Account B → Same Result
Account C → Same Result
```

With unique salts:

```text id="g2q8vm"
Account A → Salt A → Result A
Account B → Salt B → Result B
Account C → Salt C → Result C
```

An attacker recovering the password for one record still needs to perform the appropriate verification process against other salted records.

---

# Common Mistakes

### Reusing the Same Salt

Using one global salt for every password reduces the benefits of unique per-record salts.

### Using Predictable Salts

A salt should be generated from a secure random source rather than from predictable information such as:

```text id="xq3f2m"
Username
Email address
Timestamp
User ID
```

### Keeping the Salt Secret

A salt is not intended to be a password or encryption key.

Its value can normally be stored alongside the derived password value.

### Using a Fast Hash

A salt does not turn a fast hash into a password KDF.

For password storage, use an appropriate password hashing or key-derivation algorithm.

---

# Research Considerations

When analyzing password storage during an authorized assessment, document:

```text id="n7d2vc"
Algorithm
Salt Present?
Salt Length
Salt Uniqueness
KDF Parameters
Password Hash Format
```

For example:

```text id="q4z8nm"
Algorithm:      [document]
Salt:           Present
Salt:           Unique per record
KDF:            [document]
Work Factor:    [document]
```

Avoid publishing real password hashes or credentials from an assessment.

Use synthetic data when creating public research examples.

---

# Summary

A salt does not make a password secret and does not directly increase password entropy.

Its primary role is to make password-derived values unique and to prevent attackers from efficiently reusing precomputed results across multiple password records.

A secure password-storage design normally combines:

```text id="e5y2zr"
Strong Password
      +
Unique Salt
      +
Password KDF
      +
Appropriate Cost
```

Salting is one component of password storage security. It should be evaluated together with the password KDF and its configuration.


