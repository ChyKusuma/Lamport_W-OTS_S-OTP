# Signing Device One-Time Password

S-OTP (Signing Device One-Time Password) is a post-quantum secure authentication mechanism designed to replace traditional time-based or counter-based OTP systems that rely on shared secrets or symmetric keys. This method is purpose-built for the quantum computing era, leveraging cryptographic primitives like Winternitz One-Time Signatures (WOTS), Lamport signatures, and Merkle trees to ensure forward secrecy and resistance against both classical and quantum adversaries.

### 🚨 The Problem with Classical OTP Systems (HOTP/TOTP)
Traditional One-Time Password (OTP) systems like HOTP (HMAC-based OTP) and TOTP (Time-based OTP) have long served as a popular second factor in multi-factor authentication. However, they show significant weaknesses in the face of modern attack techniques and the coming quantum computing era.

🧠 Modern Threats
- OTP Phishing & Bypass Attacks
  - Attackers can trick users into entering real-time OTPs into fake websites or apps. Because OTPs are just short numeric codes (typically 6 digits), they can be captured and reused instantly with minimal detection.
- Malware & Remote Access Trojans (RATs)
  - If a user’s device is infected, OTPs can be intercepted directly as they’re generated or entered—even before they expire, completely nullifying their purpose.
- Shared Secret Compromise
  - TP/TOTP relies on a shared secret between the server and the user’s device. If that secret is ever leaked—from app reverse engineering, server compromise, or cloud misconfigurations—all future OTPs can be precomputed.
-  No Device or Identity Binding
    - Traditional OTPs are not cryptographically tied to a specific user or device. This means a valid OTP generated on one phone could be used from any attacker’s device, enabling session hijacking or unauthorized access.
- Brute-Force Vulnerability
  - Most OTPs are short (6–8 digits), meaning there are only 1 million possible values in a typical TOTP setup. Attackers with access to OTP verification endpoints can brute-force codes in seconds unless rate limiting or lockout mechanisms are extremely strict.

⚛️ Quantum-Age Risks

- Grover’s Algorithm and OTP Weakness
  - HMAC-based systems rely on symmetric key cryptography (e.g., HMAC-SHA1 or HMAC-SHA256). Quantum computers could use Grover’s algorithm to halve the effective key length, making OTP derivation exponentially faster than with classical hardware.
- Offline Quantum Attacks
  - If an attacker captures network traffic or OTP-related data, they could perform offline analysis using future quantum machines to uncover shared secrets or validate OTPs without needing live access to a user’s device.
- No Forward Security or Quantum Resistance
  - Classical OTP systems were never designed with post-quantum resilience. They cannot be upgraded to resist quantum attacks without replacing the underlying protocol and infrastructure.

## Overview

### ✅ How S-OTP Solves This
S-OTP designed is for Memory Hard + Lamport OTP + Merkle Tree + Post-Quantum registration system solves the problems that plague classical OTP systems like HOTP/TOTP:

- Lamport One-Time Signatures, which are 256-bit secure, not guessable, and tied to a specific message.
  - Each OTP is cryptographically signed, not just a short code. There’s no fixed OTP length to guess.
  - Index tracking (e.g. Index: 0) ensures a one-time use of each signature and OTP.
- Create a Merkle root hash from:
  - UserID + DeviceID + Lamport Public Key + PQPublicKey + DeviceToken
  - This rootHash cryptographically ties the OTP identity to a specific device and user.
  - The deviceToken = SHA256(userID + deviceID) ensures it's non-forgeable and device-specific.
- Lamport Signatures, which are quantum-resistant because they’re based only on hash functions.
  - Stored and register a post-quantum public key `(PQPublicKey []byte)` for future extensions (e.g., using CRYSTALS-Dilithium or SPHINCS+).
  - No shared secrets—just signed hashes. Even quantum attackers can’t forge a valid signature without the one-time private key.
- No shared secret is stored. Instead:
  - Server holds the public key (via Merkle root hash and Lamport key serialization).
  - OTPs are verified by checking cryptographic signatures, not matching a secret-derived code.
  - Use deterministic key generation from the deviceToken, so private keys can be recomputed on demand—not stored insecurely.
- Lamport OTPs are one-time and message-bound.
  - Even if a signature is captured, it cannot be reused or replayed.
  - Future plans can integrate signed challenges or contextual metadata (like timestamps or location) into the Merkle leaf or the message being signed.
- Each private key in Lamport OTS is used once and then discarded.
  - Exposure of one key does not compromise previous or future OTPs.
  - Even if one OTP/signature is stolen, only that specific one is affected.
- A memory-hard function is designed to require a large amount of memory to compute. This protects against:
  - Brute-force / dictionary attacks
  - Attackers using GPU clusters or ASICs can compute billions of hashes per second. Memory-hard algorithms slow these down by requiring not just CPU cycles but also RAM, which is a more limited and expensive resource.
- ASIC Resistance
  - Custom hardware is great at fast computation, but increasing memory requirements makes it:
  - Physically harder to build fast, low-cost ASICs
  - Expensive to scale attacks economically
- Protection against parallelization
  - Memory-hardness reduces efficiency of parallel brute-force attacks, which would otherwise make guessing weak secrets like passwords practical.


