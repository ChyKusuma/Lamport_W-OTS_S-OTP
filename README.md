# Introducing [S-OTP](https://github.com/ChyKusuma/Lamport_W-OTS_S-OTP/wiki/What-is-S%E2%80%90OTP): A Modern, Post-Quantum One-Time Password System

**S-OTP** (Signing Device for OTP) is a cutting-edge One-Time Password solution designed to protect applications using **quantum-resistant cryptography**. Powered by **Winternitz One-Time Signatures (WOTS)** and **Merkle tree-based key management**, S-OTP transforms any device into a trusted OTP signing authority.

Ideal for financial platforms, e-commerce, and enterprise-grade apps, this lightweight, scalable Go package delivers secure, reliable, and post-quantum ready authentication

## 🔐 Features

### ⚛️ Quantum-Resistant Cryptography with WOTS

- **What it does**: Uses Winternitz One-Time Signatures (WOTS) to sign and verify OTPs per device.
- **Why it matters**: Immune to quantum attacks, future-proofing your app.
- **Technical details**:
  - `GenerateLamportKeyPair` uses Argon2id to derive WOTS seeds.
  - WOTS uses 67 chains (64 message + 3 checksum) with a Winternitz parameter of 16.

### 🌳 Merkle Tree-Based Key Management

- **What it does**: Builds a Merkle tree from user/device/public key data to compute a unique root hash.
- **Why it matters**: Reduces storage and scales securely to millions of users.
- **Technical details**:
  - `RegisterHandler` and `VerifyHandler` use SHA-256 to build and verify Merkle roots.

### 🔁 Secure OTP Generation with SHAKE256 and Nonce Protection

- **What it does**: Creates OTPs using Merkle root + nonce + timestamp, hashed via SHAKE256.
- **Why it matters**: Non-reusable, non-guessable OTPs with high entropy.
- **Technical details**:
  - `GenerateOTP` outputs 6–10 char OTPs from a curated charset.
  - Nonces are 16-byte random strings stored with expiration.

### 🔐 Argon2id-Based Key Derivation

- **What it does**: Uses Argon2id for memory-hard seed derivation.
- **Why it matters**: Protects keys from brute-force even if inputs are leaked.
- **Technical details**:
  - `SelectArgon2Params` adjusts memory/iterations for mobile vs desktop.
  - `Argon2idHash` produces 32-byte key seeds.

### 📱 User Registration and Device Binding

- **What it does**: Registers each device with a unique token and binds it to a user and post-quantum key.
- **Why it matters**: Prevents unauthorized devices from generating OTPs.
- **Technical details**:
  - Device tokens are SHA-256(userID + deviceID).
  - Stored in `keydb`, returned via JSON.

### ✅ Reliable OTP Verification with Transaction Integrity

- **What it does**: Validates OTPs using stored nonces, expiration, and WOTS signature.
- **Why it matters**: Prevents race conditions and double-use.
- **Technical details**:
  - Retries failed DB transactions 3 times with 100ms backoff.
  - Marks OTPs as "burned" after use.

### ⚙️ Scalable and Concurrent-Safe Design

- **What it does**: Uses mutexes to manage concurrent operations.
- **Why it matters**: Ensures thread-safe access in high-traffic systems.
- **Technical details**:
  - `rootHashMutex` and `otpRootHashMutex` guard shared data in handlers.

### 🌐 HTTP API for Integration

- **What it does**: RESTful endpoints for registration, OTP generation, and verification.
- **Why it matters**: Works seamlessly with web, mobile, and microservices.
- **Technical details**:
  - POST-only endpoints with JSON payloads and meaningful HTTP codes.

### ⏱️ Configurable Expiration and Optimization

- **What it does**: Customizable OTP expiration and platform-aware cryptographic parameters.
- **Why it matters**: Balances UX with security on mobile and server environments.
- **Technical details**:
  - `GetExpirationTimes` computes expiry timestamps.
  - Argon2id settings adjusted via `SelectArgon2Params`.

### 🛡️ Future-Proof with Post-Quantum Public Key Support

- **What it does**: Accepts post-quantum keys at registration and verification stages.
- **Why it matters**: Keeps your system aligned with emerging crypto standards.
- **Technical details**:
  - `RegisterRequest` and `VerifyRequest` support PQ public keys.
  - Integrated into Merkle tree validation.

## 🚀 Why Choose S-OTP?

- **Uncompromising Security**: Combines WOTS, Argon2id, SHAKE256, and Merkle trees.
- **Device-Centric Design**: Every device becomes a signing authority.
- **Scalable Performance**: Thread-safe and transaction-aware for massive scale.
- **Developer-Friendly**: Simple HTTP APIs, clean JSON outputs.
- **Future-Proof**: Supports post-quantum cryptography from day one.
- **Versatile Applications**: 2FA, transaction signing, device auth, and more.


> S-OTP is more than a security library—it's a foundation for **next-gen authentication**. Empower your users with confidence, and equip your apps for the future with S-OTP.

