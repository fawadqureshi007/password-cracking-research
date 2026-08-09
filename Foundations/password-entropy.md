# Password Entropy

Password entropy is a way of describing the size of the possible password space. In password-cracking research, it helps estimate how difficult a password may be to recover when the password is generated randomly.

Entropy is normally expressed in **bits**.

## Basic Calculation

If a password is selected uniformly at random from a character set of size `N`, and the password has length `L`, the theoretical entropy is:

```text id="y7v2kd"
Entropy = L × log₂(N)
```

For example, a randomly generated password using 26 lowercase letters has:

```text id="f6p3sa"
N = 26
L = 8

Entropy = 8 × log₂(26)
        ≈ 37.6 bits
```

This is a theoretical value based on the assumption that every character is selected independently and uniformly.

Real passwords often do not satisfy those assumptions.

---

# Character Set Size

The theoretical search space depends on the number of possible characters.

Common character sets include:

| Character Set                     | Size |
| --------------------------------- | ---: |
| Lowercase letters                 |   26 |
| Uppercase letters                 |   26 |
| Digits                            |   10 |
| Lowercase + uppercase             |   52 |
| Letters + digits                  |   62 |
| Letters + digits + common symbols |  ~94 |

For a fixed password length, increasing the available character set increases the theoretical search space.

---

# Search Space

The total number of possible passwords can be expressed as:

```text id="u1q5ke"
Search Space = N^L
```

Where:

* `N` = number of possible characters
* `L` = password length

For example, an 8-character password using 10 digits has:

```text id="p9x2mw"
10^8
=
100,000,000
```

possible combinations.

An attacker performing a complete brute-force search would have to cover this space.

---

# Entropy and Cracking

Entropy is useful when evaluating a true random password because the attacker has little information about how it was generated.

For a uniform random password:

```text id="s8n4cx"
Higher Entropy
      ↓
Larger Search Space
      ↓
More Candidate Tests
      ↓
Higher Recovery Cost
```

However, entropy calculations become misleading when applied to human-created passwords without considering how the password was actually chosen.

---

# Human Passwords

Humans rarely choose passwords uniformly at random.

A password may contain:

```text id="k7v2qs"
Name
+
Year
+
Number
+
Symbol
```

For example:

```text id="w6m3pz"
ExampleName2026!
```

The theoretical character space may appear large, but an attacker does not necessarily need to search every possible 13-character combination.

If the attacker knows the password follows a predictable structure, the candidate space can be reduced considerably.

---

# Effective Search Space

This leads to an important distinction:

### Theoretical Search Space

The total number of possible strings under a defined character set and length.

### Effective Search Space

The candidates that are actually plausible given information about how the password was generated.

Consider:

```text id="e3x8vf"
Name + Year + Symbol
```

Instead of searching every possible character combination, a targeted attack can generate candidates matching that structure.

Conceptually:

```text id="d2n7ka"
Known Pattern
     ↓
Candidate Generation
     ↓
Reduced Search Space
     ↓
Password Test
```

This is why password-pattern analysis is important in red-team research.

---

# Entropy vs Length

Longer passwords generally provide a larger search space when the additional characters are genuinely unpredictable.

However:

```text id="w0k4sm"
Length ≠ Automatically Random
```

A long predictable password can have a much smaller effective search space than its raw length suggests.

For example, a password constructed from several predictable words may be easier to target than a shorter randomly generated password.

---

# Random Passwords

Random generation is important because it removes predictable human patterns.

Compare:

```text id="j8y4qn"
Summer2026!
```

with:

```text id="c2w6pk"
r7Qm2Lx9Vt4P
```

The first contains obvious semantic and structural information.

The second is intended to be random.

The actual security of either password depends on how it was generated, its length, and the attacker's available information.

---

# Diceware and Passphrases

Entropy can also be increased through randomly selected words.

A passphrase generated from a sufficiently large word list can provide substantial entropy while remaining easier for a human to remember.

The important factor is **random selection**, not simply combining words manually.

For example:

```text id="m4z7cx"
random-word
+
random-word
+
random-word
+
random-word
```

is fundamentally different from choosing four personally meaningful words.

---

# Entropy and Attack Techniques

Different attack strategies make use of different assumptions.

### Brute Force

Attempts a defined character space.

```text id="r3y8vn"
Character Set
     ↓
All Combinations
     ↓
Candidate Testing
```

### Dictionary Attack

Uses a collection of likely passwords or words.

```text id="u8p1cz"
Wordlist
   ↓
Candidate Testing
```

### Rule-Based Attack

Transforms known words into likely password variants.

```text id="k6x2sd"
Word
 ↓
Mutation Rules
 ↓
Candidates
```

### Mask Attack

Uses a known or suspected structure.

```text id="v9q4nm"
Known Pattern
     ↓
Candidate Generation
```

The more accurately the attack model describes the password-generation process, the smaller the practical candidate space may become.

---

# Entropy and Red-Team Research

When evaluating password security, do not report only the theoretical entropy.

Record the assumptions used to calculate it.

For example:

```text id="q2x7mf"
Password Length:      12
Character Set:        62
Generation Method:    Random
Theoretical Entropy:  ~71 bits
```

For a human-generated password, additional information may be more useful:

```text id="t5k9az"
Length:               12
Observed Pattern:     Word + Year + Symbol
Dictionary Presence:  Yes
Predictability:       High
```

This provides a more realistic assessment of the password's practical resistance to targeted attacks.

---

# Important Limitations

Entropy calculations can be misleading when:

* Characters are not selected randomly
* Passwords contain dictionary words
* Users reuse passwords
* Password patterns are known
* Attackers have contextual information
* Passwords are generated from predictable templates

A mathematical entropy estimate is therefore not automatically equivalent to real-world password strength.

---

# Summary

Password entropy describes the uncertainty involved in selecting a password.

For uniformly random passwords:

```text id="b8w3xp"
Entropy = L × log₂(N)
```

But real-world password security depends on how the password was generated.

For red-team research, the useful question is not only:

> "How many possible characters are there?"

It is also:

> "How predictable is the way this password was created?"

That distinction becomes important when comparing brute-force, dictionary, rule-based, mask, and targeted password-recovery techniques.


