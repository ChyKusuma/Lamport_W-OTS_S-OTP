# A Modern, Post-Quantum One-Time Password System

# Directory
* [Introduction](https://github.com/ChyKusuma/Lamport_W-OTS_S-OTP/wiki/What-is-S%E2%80%90OTP)
* [Register models](https://github.com/ChyKusuma/Lamport_W-OTS_S-OTP/wiki/Register-Handler)
* [Request models](https://github.com/ChyKusuma/Lamport_W-OTS_S-OTP/wiki/Request-Handler)
* [Signature models](https://github.com/ChyKusuma/Lamport_W-OTS_S-OTP/wiki/Winternitz-One%E2%80%90Time-Signature)
* [Verification  models](https://github.com/ChyKusuma/Lamport_W-OTS_S-OTP/wiki/Verify-Handler)
* [Cryptography](https://github.com/ChyKusuma/Lamport_W-OTS_S-OTP/wiki/One%E2%80%90time-key-pair-generation)

# Summary
Traditional TOTP (Time-based One-Time Passwords) which relies on symmetric and with or not used asymetric primitive cryptographic keys like this can be vulnerable to interception, bypass, or even derivied corrected private key from public key, especially in a future with powerful quantum computers.

![registration process](https://github.com/ChyKusuma/Lamport_W-OTS_S-OTP/blob/main/.github/workflows/classical%20otp/classicOTP.png)
