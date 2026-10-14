# 🌸 Ethereum Cryptographic Pipeline & Key Derivation
### Homework 2 — Advanced Cryptography & Statistics for Distributed Ledgers

<p align="left">
  <img src="https://img.shields.io/badge/Domain-Cybersecurity%20%26%20Blockchain-ff7597?style=for-the-badge&logo=ethereum&logoColor=white" />
  <img src="https://img.shields.io/badge/Curve-secp256k1-fda4af?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Hash-Keccak--256-f43f5e?style=for-the-badge" />
</p>

---

## 🎀 Architecture Overview: The Key Derivation Pipeline

The transition from a human-readable seed phrase to an on-chain Ethereum account address follows a strictly deterministic, multi-primitive sequence:

```text
[ BIP-39 Mnemonic Phrase (12/24 words) ]
                   │
                   ▼  PBKDF2 (HMAC-SHA512, 2048 rounds, salt="mnemonic"+passphrase)
[ 512-bit Binary Master Seed ]
                   │
                   ▼  BIP-32 / BIP-44 Derivation Path: m/44'/60'/0'/0/0
[ 256-bit Private Scalar Key (k) ]
                   │
                   ▼  secp256k1 Scalar Point Multiplication: Q = k · G
[ 64-byte Uncompressed Public Key (X, Y) ]
                   │
                   ▼  Keccak-256 (256-bit Sponge Hash Digest)
[ 32-byte Hash Output ]
                   │
                   ▼  Truncation: Drop upper 12 bytes, retain lower 20 bytes (160 bits)
[ 20-byte Raw Ethereum Hex Address ]
                   │
                   ▼  EIP-55 Mixed-Case Capitalization Checksum
[ 0x5aAeb6053F3E94C9b9A09f33669435E7Ef1BeAed ]
