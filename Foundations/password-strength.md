# Password Strength

Password strength describes how difficult a password is to guess or recover.

Length is an important factor, but it is not the only factor. Password generation method, predictability, reuse, known patterns, and attacker knowledge can significantly change the practical difficulty of recovering a password.

## Length

A longer password generally provides a larger search space when the characters are unpredictable.

For example:

```text id="k3v8px"
Short
  ↓
Smaller Search Space

Longer
  ↓
Larger Search Space
```

However, adding predictable characters does not necessarily provide the same security benefit as adding genuinely random characters.

---

## Randomness

A randomly generated password is generally harder to target than a password created from predictable information.

Compare:

```text id="p7m2qd"
Pakistan2026!
```

with:

```text id="n4x8zs"
q7V!m2Kp9#Lx
```

The first contains recognizable words and a predictable year pattern.

The second is intended to be randomly generated.

The difference is not simply the number of characters. It is also the amount of information an attacker can infer about how the password was created.

---

# Predictable Patterns

Human-created passwords frequently contain patterns such as:

```text id="v5k9cx"
Word + Number
Word + Year
Name + Date
Word + Symbol
Word + Word + Number
Capitalized Word + Digits
```

Examples:

```text id="z8q3mw"
Summer2026
Admin123
Password!
Company2026
Welcome01
```

These patterns are commonly represented in password dictionaries and mutation rules.

A password that appears complicated may therefore still have a relatively small effective candidate space.

---

# Dictionary Exposure

A password may be technically long but still predictable if it is composed of common words.

For example:

```text id="c4x7qn"
correcthorsebatterystaple
```

Length alone does not determine whether a password is resistant to targeted guessing.

An attacker may use dictionaries containing:

* Common passwords
* Dictionary words
* Names
* Places
* Companies
* Sports teams
* Common phrases
* Previously exposed passwords

The contents of the candidate list should be considered when evaluating a password's practical strength.

---

# Password Reuse

Password reuse creates additional risk.

Consider a user who uses the same password for:

```text id="f2n8yk"
Email
VPN
SSH
Cloud Service
Internal Application
```

If the password is compromised at one service, it may be useful against the others.

This is especially important during authorized red-team assessments because credential reuse can turn a single compromised credential into access to multiple systems.

---

# Contextual Information

Attackers may have information about the target that reduces the candidate space.

Examples include:

```text id="q7m3dv"
Username
Company Name
Domain Name
Location
Pet Name
Team Name
Product Name
Important Dates
```

For example, if an organization's password policy allows:

```text id="w6p9sx"
CompanyName + Year + Symbol
```

then testing that structure may be more efficient than testing every possible password of the same length.

---

# Password Policy

Password policies can affect both password security and user behavior.

A policy may require:

* Minimum length
* Uppercase characters
* Lowercase characters
* Numbers
* Special characters
* Password expiration
* Password history

Complexity requirements alone do not guarantee strong passwords.

A policy that forces predictable transformations can sometimes produce passwords such as:

```text id="d4y8rm"
Password1!
Password2!
Password3!
```

A longer minimum password length combined with support for strong randomly generated passwords is generally more useful than relying only on character-class requirements.

---

# Password Strength and Attack Techniques

Different password characteristics favor different candidate-generation strategies.

| Password Characteristic    | Relevant Technique  |
| -------------------------- | ------------------- |
| Common password            | Dictionary          |
| Known word with mutations  | Rules               |
| Known structure            | Mask                |
| Small character space      | Brute force         |
| Known personal information | Targeted candidates |
| Multiple known patterns    | Hybrid approach     |

This does not mean a specific technique will always succeed. It describes why password characteristics influence attack selection.

---

# Measuring Password Strength

During a controlled assessment, useful observations include:

```text id="m8q2vf"
Length
Character Composition
Dictionary Presence
Known Pattern
Predictability
Password Reuse
Estimated Candidate Space
Observed Recovery Result
```

A useful assessment should explain **why** a password was recoverable rather than simply stating that it was cracked.

For example:

```text id="h3x7kp"
Finding:
Password followed a predictable word + year pattern.

Impact:
The effective candidate space was significantly smaller
than a full brute-force search of the same length.

Recommendation:
Use unique, randomly generated passwords and avoid
predictable organizational patterns.
```

---

# Strong Password Characteristics

A strong password should generally be:

* Long
* Unique
* Randomly generated where practical
* Not based on personal information
* Not reused across services
* Resistant to common dictionary and pattern-based attacks

For human-memorable passwords, a randomly generated passphrase can provide a useful balance between usability and security.

For systems that support password managers, randomly generated passwords can provide substantially larger candidate spaces than typical human-created passwords.

---

# Password Managers

Password managers can help users maintain unique passwords without requiring them to memorize every password.

A typical workflow is:

```text id="r6m9xc"
Password Manager
      ↓
Generate Random Password
      ↓
Store Unique Credential
      ↓
Use Different Password Per Service
```

This reduces the impact of password reuse.

The security of the password manager itself must also be considered, particularly its authentication and encryption mechanisms.

---

# Research Notes

When documenting password-strength research, avoid publishing real credentials.

Use synthetic examples:

```text id="v2q5nm"
ExamplePassword123!
```

rather than credentials obtained from real users or unauthorized systems.

For each experiment, record the password-generation assumptions and the attack technique used.

Example:

```text id="p8k3yw"
Password Type:      Synthetic
Length:             10
Pattern:            Word + Year + Symbol
Attack:             Dictionary + Rules
Result:             Recovered
```

This makes the experiment reproducible without exposing sensitive information.

---

# Summary

Password strength is determined by more than length or character complexity.

A practical assessment should consider:

```text id="s4x8qd"
Length
   +
Randomness
   +
Predictability
   +
Uniqueness
   +
Attacker Knowledge
   +
Storage Protection
```

A long password can still be weak if its construction is predictable.

For password-cracking research, understanding how passwords are generated is often as important as understanding the cracking tool itself.

