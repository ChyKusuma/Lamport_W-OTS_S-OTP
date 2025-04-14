# Signing Device method for One-Time Password

### Introducing [S-OTP](https://github.com/ChyKusuma/Lamport_W-OTS_S-OTP/wiki/W%E2%80%90OTS): A Modern, Post-Quantum One-Time Password System

>### From [Leslie Lamport, 1979](https://www.microsoft.com/en-us/research/publication/constructing-digital-signatures-one-way-function/) to the Quantum World.

S-OTP (Signing Device One-Time Password) is a next-generation authentication system built to be quantum-secure, cryptographically signed, non-replayable, and memory-hard to attack—without relying on shared secrets or symmetric keys.

Unlike traditional OTP systems that use time or counters with a shared secret, S-OTP takes a fundamentally different approach. It generates OTPs from digital signatures using [Winternitz One-Time Signatures (WOTS)](https://en.wikipedia.org/wiki/Hash-based_cryptography)—a hash-based signature scheme proven to be secure even against quantum computers.

Each user’s identity (userID) and device fingerprint (deviceID) are combined and used as a deterministic seed to derive their private signing keys. These keys are then organized into a Merkle tree, allowing many one-time signatures to be verified using a single, reusable Merkle root hash. This setup ensures both forward secrecy and efficient key management without requiring storage of every public key.

We call it S-OTP because it leverages a Signing Device to produce One-Time Passwords—OTP codes that are not only tied to the device and user, but also signed cryptographically, making them tamper-proof, non-replayable, and independently verifiable.

By blending post-quantum cryptography, deterministic key generation, and stateless verification, S-OTP represents a forward-looking, secure, and scalable solution for authentication in the age of quantum computing.
