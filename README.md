# Signing Device One-Time Password

From Leslie Lamport to Quantum world. S-OTP (Signing Device One-Time Password) is a post-quantum secure authentication mechanism designed to replace traditional time-based or counter-based OTP systems that rely on shared secrets or symmetric keys. This method is purpose-built for the quantum computing era, leveraging cryptographic Winternitz One-Time Signatures (WOTS), Lamport signatures, and Merkle trees to ensure forward secrecy and resistance against both classical and quantum adversaries.

# Background

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

### ✅ How This `otp` Package Solves Modern OTP Challenges
This package implements a **Post-Quantum Secure, Memory-Hard, and Replay-Proof OTP system**
built on Winternitz-based Lamport One-Time Signatures and Merkle trees.

🔐 Core Features:
- **Winternitz One-Time Signatures (WOTS)**:
   - Each OTP is signed with a cryptographic signature derived from WOTS, not just a guessable code.
   - The Winternitz base-w parameter improves efficiency over classic Lamport signatures.
   - OTPs are bound to a message via SHAKE256 hashes, preventing replay attacks.
   - Signatures are unforgeable without the private key, and each key is used once.
     
- **Merkle Tree Root as Public Key**:
  - The public key segments (WOTS) are committed to a Merkle tree to derive a reusable `rootHash`.
  - This root hash binds: UserID + DeviceToken + WOTS Public Key → ensuring strong user/device identity.
  - The root serves as the system’s trust anchor for verifying OTPs.
  
- **Memory-Hard Argon2id Key Derivation**:
   - Used to derive deterministic WOTS private keys from `(userID, deviceToken)` input.
   - Prevents brute-force or dictionary attacks by requiring high memory cost to compute.
   - Parameters adapt to platform (mobile vs server) via `SelectArgon2Params()`.

- **No Shared Secrets Stored**:
   - The server stores only public values (Merkle root, public keys).
   - Private keys are generated on-demand, never stored.
   - No TOTP-like shared secrets that could leak or be brute-forced.

- **Replay and Forgery Resistance**:
  - Each OTP is used once, bound to a hash of the message.
  - Once used, the key segment is discarded — protecting future and past signatures even if compromised.
    
- **Post-Quantum Secure**:
  - All signature operations use only SHAKE256 (hash-based), making them quantum-resistant.
  - Ideal for secure OTP systems in the post-quantum era.

- **Extensible Design**:
  - Easily supports additional metadata (e.g., timestamp, geolocation) in the signed message.
  - Optional Merkle path authentication and future Dilithium/SPHINCS+ PQ key integration.

📦 Main Components in This Design :
  - `GenerateLamportKeyPair`: Derives WOTS private/public key pairs using Argon2id and SHAKE256.
  - `Sign`: Signs a message using base-w WOTS with checksum for message integrity.
  -  `Argon2idHash` and `VerifyArgon2idHash`: For memory-hard password hashing and authentication.
  - `GetExpirationTimes`: Standardized timestamp management for OTP lifecycle.

✅ Result: A modern OTP system that is cryptographically signed, quantum-secure, non-replayable, memory-hard to attack, and doesn't rely on shared secrets.


