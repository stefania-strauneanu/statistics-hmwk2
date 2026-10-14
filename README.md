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
Cryptographic Primitives & Standards1. BIP-39 — Mnemonic Sentence & Master Seed GenerationEntropy to Words: Maps 128–256 bits of CSPRNG entropy into 12 or 24 human-memorizable tokens selected from the fixed 2048-word dictionary, terminating in an appended checksum bit sequence.Key Stretching: Applies PBKDF2 configured with HMAC-SHA512 over 2,048 computational iterations. The salt is constructed dynamically as UTF8("mnemonic" + optional_passphrase). This expands entropy to a 512-bit master seed while resisting brute-force offline dictionary attacks.2. BIP-32 — Hierarchical Deterministic (HD) Key DerivationTree Structure: Enables a single master seed to compute an infinite directed tree of key pairs without re-seeding.Child Derivation Function (CKD): Child keys are evaluated using HMAC-SHA512(ChainCode, ParentPubKey || Index).Hardened vs Non-Hardened: Indices $\ge 2^{31}$ force hardened derivation, executing HMAC against the parent private key to stop leakage propagation up the tree.3. BIP-44 — Multi-Account Derivation PathsStandardizes organized path routing for multi-currency HD wallets:$$\text{m} \;/\; \text{purpose}' \;/\; \text{coin\_type}' \;/\; \text{account}' \;/\; \text{change} \;/\; \text{address\_index}$$For Ethereum Mainnet: m / 44' / 60' / 0' / 0 / 0 (where 60' denotes ETH).4. secp256k1 — Elliptic-Curve CryptographyParameters: A Koblitz curve over $\mathbb{F}_p$ where $p = 2^{256} - 2^{32} - 977$, defined by $y^2 \equiv x^3 + 7 \pmod p$.One-Way Mapping: Public key $Q = k \cdot G$ maps scalar $k \in [1, n-1]$ to a point on the curve. Forward multiplication is $O(\log k)$ using Double-and-Add; backward inversion requires solving the Elliptic Curve Discrete Logarithm Problem (ECDLP), providing $\approx 128$ bits of symmetric security.5. Keccak-256 — Cryptographic Sponge HashingEvaluates the serialized 64-byte $(X, Y)$ coordinate representation of public key point $Q$.The raw address strips away the first 12 bytes and retains the lower 20 bytes (160 bits).6. EIP-55 — Mixed-Case Checksum EncodingPrevents silent failure when typing or copying addresses.For each hex character $A_i$ in lowercase address $A$:$$\text{If } A_i \in [\text{a-f}] \text{ and the } i\text{-th nibble of } \text{Keccak-256}(A) \ge 8 \implies \text{Uppercase}(A_i)$$🧠 Core Architectural DistinctionsConceptPrimitivePrimary ObjectiveKey GenerationCSPRNG Hardware EntropySamples high-entropy bitstreams to bootstrap identity.Key DerivationPBKDF2 / HMAC-SHA512Expands and derives deterministic keys from a single seed.Public-Key Cryptosecp256k1Provides asymmetric digital signature verification (ECDSA).HashingKeccak-256Compresses state, eliminates algebraic group structures.EncodingHexadecimal (0x)Translates binary byte arrays into readable strings.ChecksumEIP-55Integrates in-place error detection without changing payload length.⏳ Chronological Evolution1977 — RSA: Landmark public-key system relying on integer factorization hardness.2001 — SHA-2 (SHA-512): Standardized by NIST; iterative Merkle-Damgård block compression.2012 — BIP-32: Hierarchical Deterministic wallet specification introduced for Bitcoin.2013 — BIP-39: Standardization of mnemonic seed sentences.2014 — BIP-44: Unified multi-account derivation tree standard.2015 — SHA-3: NIST standardization of the Keccak sponge function family.2016 — EIP-55: Vitalik Buterin introduces backward-compatible checksum encoding.🔍 Why Ethereum's Keccak-256 differs from official NIST SHA3-256:Ethereum incorporated Keccak in late 2014 prior to final NIST ratification. The final NIST FIPS 202 standard added domain separation bytes (0x06 for SHA3-256 vs. 0x01 in raw Keccak). Hence, Keccak-256 and SHA3-256 yield different digest outputs.📊 Statistical Analysis & Security BoundsEntropy Propagation & Uniformity: Outputs from HMAC-SHA512 and Keccak-256 satisfy the Strict Avalanche Criterion (SAC); changing 1 input bit alters output bits with probability $P \approx 0.5$.Truncation Collision Bounds (Birthday Paradox): Truncating Keccak-256 to 160 bits ($20$ bytes) lowers the collision bound from $2^{128}$ to $2^{80}$ operations. In distributed ledger architecture, $2^{80}$ operations remains computationally unfeasible while reducing storage on the state trie.EIP-55 Typo Detection: With an average of 15–20 alphabetic characters per address, the probability of an undetected accidental typo is $P(\text{undetected}) \approx 2^{-15} \approx 0.003\%$, yielding an empirical error-detection rate of $99.997\%$.
