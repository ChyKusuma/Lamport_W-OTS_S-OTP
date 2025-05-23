# Introducing [S-OTP](https://github.com/ChyKusuma/Lamport_W-OTS_S-OTP/wiki/What-is-S%E2%80%90OTP): A Modern, Post-Quantum One-Time Password System

**S-OTP** (Signing Device for OTP) is a cutting-edge One-Time Password solution designed to protect applications using **quantum-resistant cryptography**. Powered by **Winternitz One-Time Signatures (WOTS)** and **Merkle tree-based key management**, S-OTP transforms any device into a trusted OTP signing authority.

Ideal for financial platforms, e-commerce, and enterprise-grade apps, this lightweight, scalable Go package delivers secure, reliable, and post-quantum ready authentication

```bash
Client                       Server
  |                            |
  | --- POST /request_otp ---->|  (RequestOTPHandler)
  |  {user_id, device_token,   |
  |   pub_key, signature,      |
  |   timestamp}               |
  |                            |  1. Validate POST method
  |                            |  2. Decode JSON into OTPRequest
  |                            |  3. Validate timestamp (±5 minutes)
  |                            |  4. Fetch user by UserID (keydb.Get)
  |                            |  5. Validate DeviceToken (ValidateDeviceToken)
  |                            |     a. Fetch user from keydb
  |                            |     b. Recompute DeviceID using GetDeviceID
  |                            |     c. Verify DeviceID matches stored
  |                            |     d. Recompute DeviceToken (SHA3-256: UserID, DeviceID, ServerSecret)
  |                            |     e. Compare stored, recomputed, provided DeviceTokens
  |                            |     f. Clean up sensitive data (ZeroBytes)
  |                            |  6. Validate Public Key
  |                            |     a. Decode provided pub_key (hex)
  |                            |     b. Compare with stored PubKey
  |                            |  7. Verify Signature
  |                            |     a. Decode signature
  |                            |     b. Verify signature with pub_key (otp.Verify: user_id:device_token:timestamp)
  |                            |  8. Validate OTP (ValidateOTP)
  |                            |     a. Check for unexpired/active OTP (otpdb.GetOTP)
  |                            |     b. Return conflict if active OTP exists
  |                            |  9. Generate OTP
  |                            |     a. Randomly select length (6, 8, or 10 digits)
  |                            |     b. Generate OTP and nonce (otp.GenerateOTP)
  |                            | 10. Store OTP (otpdb.StoreOTPWithExpiry)
  |                            |     a. Store OTP, nonce, signature, pub_key, issued_at, expires_at
  |                            | 11. Log OTP generation
  |                            | 12. Clean up sensitive data (ZeroBytes)
  | <--- 200 OK -------------- |  {status: "OTP issued", otp, nonce, expires_at, issued_at, message}
  |  {status, otp, nonce,      |
  |   expires_at, issued_at,   |
  |   message}                 |
```

## 🔐 Pros and Cons Comparison

| **Aspect**              | **SOTP**                                                                                   | **TOTP**                                                                                      | **HOTP**                                                                                      |
|-------------------------|---------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| **Security Model**      | Lamport signatures (hash-based, post-quantum secure), no shared secret.                          | HMAC-based with shared secret, time-based.                                                    | HMAC-based with shared secret, counter-based.                                                 |
| **Pros**                | - Post-quantum resistant. <br> - No shared secret, reducing server breach impact. <br> - Nonce prevents replay attacks. <br> - Device token adds authentication layer. <br> - One-time signatures prevent reuse. | - Simple and standardized (RFC 6238). <br> - Widely supported (e.g., Google Authenticator). <br> - Offline OTP generation. <br> - Short validity window (30s). | - Simple and standardized (RFC 4226). <br> - No time synchronization needed. <br> - Suitable for hardware tokens. <br> - Offline OTP generation. |
| **Cons**                | - Complex implementation (Lamport signatures, Merkle trees, database transactions). <br> - Client must manage multiple private keys. <br> - Unverified signature storage in `/request_otp`. <br> - Server-side key generation for unsigned OTPs. <br> - Short 30s window may cause usability issues. | - Shared secret vulnerable to server breaches. <br> - No inherent replay protection. <br> - Time synchronization required. <br> - Not post-quantum secure (symmetric key halved by Grover’s algorithm). | - Shared secret vulnerable to server breaches. <br> - Counter synchronization issues. <br> - No time-based expiration increases interception risk. <br> - Not post-quantum secure. |
| **Attack Surface**      | - Implementation errors due to complexity. <br> - Server-side random number generation for new signatures. <br> - Database integrity for OTP storage. | - Server-side secret storage. <br> - Interception within time window. <br> - Phishing if OTP is entered on malicious site. | - Server-side secret storage. <br> - Counter desynchronization. <br> - Interception without time limit. |
| **Usability**           | Moderate: Requires client to manage keys and signatures, short OTP validity may cause timeouts.   | High: Simple for users, supported by standard apps, but requires time sync.                   | High: Simple for users, no time sync needed, but counter issues may lock out users.           |
| **Scalability**         | High: Merkle tree supports large user bases, but key management is complex.                      | High: Simple server-side computation, but secret storage scales linearly.                     | High: Simple computation, but counter management scales linearly.                             |
| **Post-Quantum Security** | ✅ Yes: Hash-based signatures are quantum-resistant.                                              | ⚠️ Partial: Symmetric HMAC is affected by Grover’s algorithm, but impact is limited.          | ⚠️ Partial: Same as TOTP.                                                                     |
| **Implementation Complexity** | 🔺 High: Requires cryptographic expertise for Lamport signatures, Merkle trees, and secure key management. | ✅ Low: Standardized libraries available, minimal setup.                                      | ✅ Low: Standardized libraries, but counter sync adds complexity.                              |


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

- **High-Level Security**: Combines WOTS, Argon2id, SHAKE256, and Merkle trees.
- **Device-Centric Design**: Every device becomes a signing authority.
- **Scalable Performance**: Thread-safe and transaction-aware for massive scale.
- **Developer-Friendly**: Simple HTTP APIs, clean JSON outputs.
- **Future-Proof**: Supports post-quantum cryptography from day one.
- **Versatile Applications**: 2FA, transaction signing, device auth, and more.


> S-OTP is more than a security library—it's a foundation for **next-gen authentication**. Empower your users with confidence, and equip your apps for the future with S-OTP.

