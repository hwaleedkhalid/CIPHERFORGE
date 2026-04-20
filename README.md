# CipherForge — Encryption Techniques Report

 Web-Based Text Encryption Tool 

---

## Overview

CipherForge implements seven cryptographic algorithms spanning three categories: classical substitution ciphers, modern symmetric encryption, and one-way hash functions. All operations execute entirely in the browser using JavaScript and the CryptoJS library — no plaintext or keys are transmitted over any network.

---

## 1. Classical Ciphers

### 1.1 Caesar Cipher

The Caesar cipher is one of the oldest and simplest encryption techniques, historically attributed to Julius Caesar who reportedly used a shift of three to protect military communications. It operates by shifting each letter of the alphabet by a fixed integer value known as the *shift key*.

**How it works:** Each alphabetic character is mapped to a new character by adding the shift value to its position in the alphabet, wrapping around modularly. For example, with a shift of 3, the letter `A` becomes `D`, `B` becomes `E`, and `Z` wraps around to `C`. Non-alphabetic characters such as digits, spaces, and punctuation are left unchanged.

**Decryption** is simply the inverse operation — subtracting the same shift. Because there are only 25 possible non-trivial shifts, the cipher is entirely vulnerable to brute-force attack. Its value today is educational rather than practical.

**Key type:** Integer (1–25)  
**Security level:** Very low — broken by exhaustive search in milliseconds

---

### 1.2 Vigenère Cipher

The Vigenère cipher is a polyalphabetic substitution cipher that extends the Caesar concept by using a *keyword* instead of a single shift. Each letter of the keyword determines a different shift for the corresponding letter of the plaintext, cycling through the keyword repeatedly for messages longer than the key.

**How it works:** The keyword is converted to a sequence of numeric shifts (A=0, B=1, ... Z=25). The first character of the plaintext is shifted by the first character of the key, the second by the second, and so on. This means the same plaintext letter can encrypt to different ciphertext letters depending on its position — a significant improvement over monoalphabetic ciphers.

**Example:** With keyword `KEY` and plaintext `HELLO`, the shifts are 10, 4, 24, 10, 4, producing `RIJVS`.

For centuries this cipher was considered unbreakable, earning the nickname *"le chiffre indéchiffrable"* (the indecipherable cipher). It was eventually broken in the 19th century using frequency analysis techniques such as the Kasiski test, which exploit the repeating key pattern.

**Key type:** Alphabetic keyword (longer = stronger)  
**Security level:** Low by modern standards, but far stronger than Caesar

---

### 1.3 ROT-13

ROT-13 ("rotate by 13") is a special case of the Caesar cipher with a fixed shift of 13. Because the English alphabet has 26 letters, applying ROT-13 twice restores the original text — the encryption and decryption operations are identical. This self-inverse property makes it a toggle rather than a true cipher.

**How it works:** Each letter is shifted by exactly 13 positions. `A` becomes `N`, `N` becomes `A`, `Z` becomes `M`, and so on. Digits and special characters are not altered.

ROT-13 provides no security whatsoever. Its primary real-world use is hiding spoilers in online forums and obfuscating text in contexts where the goal is mild concealment rather than genuine confidentiality.

**Key type:** None (fixed shift of 13)  
**Security level:** None — intended only for casual obfuscation

---

## 2. Modern Encryption

### 2.1 AES-256 (Advanced Encryption Standard)

AES is the global standard for symmetric-key encryption, adopted by the U.S. National Institute of Standards and Technology (NIST) in 2001 and used by governments, militaries, financial institutions, and virtually all secure software worldwide. CipherForge uses the 256-bit key variant, which is the strongest configuration of AES available.

**How it works:** AES is a *block cipher* — it divides plaintext into fixed 128-bit blocks and processes each through a series of mathematical transformations across multiple rounds (14 rounds for AES-256). These transformations include byte substitution (SubBytes), row shifting (ShiftRows), column mixing (MixColumns), and key addition (AddRoundKey). The result is a ciphertext that is computationally indistinguishable from random noise without the correct key.

In CipherForge, the user's passphrase is used by CryptoJS to derive a 256-bit key through an internal key derivation process. The output ciphertext is Base64-encoded and includes an embedded salt so that identical passphrases produce different ciphertexts on each encryption — this is a critical security property called *semantic security*.

**Decryption** requires the exact same passphrase used for encryption. A wrong passphrase produces garbage output, which the application detects and reports as an error.

**Key type:** Passphrase (any length; longer and more complex = more secure)  
**Security level:** Very high — currently considered computationally unbreakable with a strong passphrase

---

### 2.2 Base64

Base64 is an encoding scheme, not an encryption algorithm. It converts arbitrary binary data into a printable ASCII string using a 64-character alphabet (A–Z, a–z, 0–9, `+`, `/`) with `=` padding. It is included in CipherForge because it is frequently encountered alongside cryptographic systems and is often mistaken for encryption by beginners.

**How it works:** Every three bytes (24 bits) of input are split into four 6-bit groups, each mapped to one of the 64 characters in the Base64 alphabet. If the input is not divisible by three, padding characters (`=`) are added to complete the final group.

**Decoding** is deterministic and requires no key — anyone with the encoded string can recover the original data instantly. Base64 is used in practice for embedding binary data in text-based protocols such as email (MIME), HTTP Authorization headers, and data URIs for images.

**Key type:** None  
**Security level:** Zero — provides no confidentiality whatsoever

---

## 3. Hash Functions

Hash functions are a fundamentally different category from the ciphers above. They are *one-way* mathematical functions: given an input, they produce a fixed-size output (the *digest* or *hash*), but it is computationally infeasible to reconstruct the input from the hash. CipherForge includes two hash functions for reference and educational purposes. The application enforces that decryption cannot be attempted on hash outputs.

---

### 3.1 SHA-256 (Secure Hash Algorithm 2)

SHA-256 is a member of the SHA-2 family of hash functions designed by the NSA and standardized by NIST. It produces a 256-bit (64 hexadecimal character) digest. It is the most widely deployed cryptographic hash function in use today, underpinning TLS/HTTPS certificates, code signing, Git commit integrity, Bitcoin proof-of-work, and the storage of passwords.

**How it works:** SHA-256 processes input in 512-bit chunks through 64 rounds of a compression function involving bitwise operations (AND, OR, XOR, NOT), modular addition, and right-rotation of 32-bit words. The algorithm maintains an internal state of eight 32-bit words initialized to constants derived from the square roots of the first eight prime numbers. The final state after processing all chunks is concatenated to form the 256-bit digest.

A fundamental property is *collision resistance* — it is computationally infeasible to find two different inputs that produce the same hash. Even a single-character change to the input produces a completely different output (the *avalanche effect*), making SHA-256 ideal for data integrity verification.

**Output length:** 256 bits / 64 hex characters  
**Security level:** High — no practical attacks known

---

### 3.2 MD5 (Message Digest 5)

MD5 was designed by Ronald Rivest in 1991 and produces a 128-bit (32 hexadecimal character) digest. It was once widely used for passwords, certificates, and file integrity, but is now considered cryptographically broken and should not be used for any security-sensitive purpose.

**How it works:** MD5 processes input in 512-bit blocks through four rounds of 16 operations each, using a combination of bitwise logic, modular addition, and left rotation on four 32-bit state words. The design is faster than SHA-256 but sacrifices security for that speed.

**Why it is broken:** Collision attacks against MD5 have been demonstrated since 2004. Researchers have shown that two different files can be crafted to produce identical MD5 hashes, completely defeating its use for certificate integrity or tamper detection. MD5 hashes of passwords can also be reversed for common inputs using precomputed *rainbow tables*. Its inclusion in CipherForge is strictly for legacy reference and educational demonstration.

**Output length:** 128 bits / 32 hex characters  
**Security level:** Low — cryptographically broken; do not use for security applications

---

## 4. Algorithm Comparison Summary

| Algorithm | Type | Reversible | Key Required | Security Level |
|---|---|---|---|---|
| Caesar Cipher | Classical substitution | Yes | Numeric shift | Very Low |
| Vigenère Cipher | Polyalphabetic substitution | Yes | Alphabetic keyword | Low |
| ROT-13 | Fixed substitution | Yes (self-inverse) | None | None |
| AES-256 | Symmetric block cipher | Yes | Passphrase | Very High |
| Base64 | Encoding | Yes | None | None |
| SHA-256 | Cryptographic hash | No | None | High |
| MD5 | Cryptographic hash | No | None | Low (broken) |

---

## 5. Security Considerations

All cryptographic operations in CipherForge are executed client-side in the browser. No plaintext, ciphertext, keys, or passphrases are sent to any server. The application uses the CryptoJS 4.2.0 library loaded from the Cloudflare CDN for AES-256, SHA-256, and MD5 operations. Classical cipher algorithms are implemented from scratch in pure JavaScript without any external dependencies.

Users should note that the security of AES-256 encryption is entirely dependent on the strength of the passphrase chosen. A weak or guessable passphrase reduces effective security regardless of the underlying algorithm's theoretical strength. For any real-world sensitive data, a strong, unique passphrase of at least 16 characters combining mixed case, numbers, and symbols is strongly recommended.

---

