# Winternitz One-Time Signature (Blake2b)

## Introduction

Winternitz One-Time Signatures are powerful, post-quantum signature schemes that offer robust security based on the hardness of finding collisions in hash functions and the hardeness of finding the preimage of hash functions. This paper presents a new method using BLAKE2 to design winternitz-ots to be even more compact. It builds on several key components of other hash-based signature functions and delivers a more compact signature size using a range of 16 bytes.

There exists a multitude of hash-based signature schemes like Lamport Signatures, Winternitz-OTS, WOTS+, XMSS+ (stateful), SPHINCS+ (stateless), that all happen to be resistant to post-quantum cryptography.

We present many findings on Hash-Based Signature Schemes that will be presented throughout this document. These findings are open to public and open to discussion.


## 1. Introduction To BLAKE2 (one of the NIST's SHA3 Finalists)

In `BLAKE2`, as specified in [RFC7693](https://datatracker.ietf.org/doc/html/rfc7693) there exists many properties that are **valuable** in the construction provided. These are:

1. **Variable-Digest Hash Lengths:** (1-64 Bytes for BLAKE2b; 1-32 bytes for BLAKE2s). These are unique per each input depending on the variable size of the output.
2. **Keyed-Hashing:** BLAKE2 allows for key hashing which allows the input of a key to be applied to the hash function, resulting in a different hash digest.
3. **Fast Hashing:** BLAKE2 offers faster hashing than most cryptographic hash functions while still providing robust security.
4. **[Security] Immunity To Length Extension Attacks:** BLAKE2 is said to be immune to **length-extension attacks** unlike SHA2.
5. **Optimized For Different Systems**: BLAKE2b is optimized for 64-bit systems while BLAKE2s is optimized for 8 to 32-bit systems.

In BLAKE2, construction of the hash function is done like the following:

### BLAKE2B

#### Params

**Bits in words:** `w = 64`

**Rounds:** `r = 12`

**Block Bytes:** `bb = 128`

**Hash Bytes:** `1 <= nn <= 64`

**Key Bytes:** `0 <= kk <= 64`

**Input Bytes:** `0 <= ll <= 2^128`

## 2. Naive Construction of WOTS-ESSIK

This is a naive construction of WOTS-ESSIK using BLAKE2 for constant public key lengths.

### Assumptions

We assume we are signing a message of 256-bits (32-bytes).

We assume we are using the `w` parameter of `16`, signing 4 bits at a time (0-15).

We assume we are using BLAKE2B with variable digest length from 32-48.

### Method 1: WOTS-Inverse (Constant Mapping of Public Keys; K-Inverse)

1. We generate a `nonce`.
2. We generate a `secret key` of 32-64 bytes using randomness.
3. We use `BLAKE2B(32-48)` to iterately hash the `secret key` keyed by `iterator xor nonce` to map out our hashes
4. We then hash each key by `message_size_in_bits / 4` which for a 256-bit message would be `64x` with BLAKE2B(variable_digest) to get the `public key`. If needed, this number can be reduced but will not be provably enough for that many iterations if the message is 32-bytes (256-bits)
5. To sign, we take the bit-length, and then use the key in order to iterate over each byte using the given hash digest size (n-1) for each byte. So to hash, we would hash until reduction of 1 from the public key until we get as many as we can. This is more secure than using other methods as only a little amount of the key is shown.

#### Advantages
- Post-Quantum One-Time Signature
- Small Secret Key Size (only requiring a `secret key` and `nonce`)
- Slightly Larger Signature Size
- Small, Constant Public Key Size
- Stronger Security Assumptions
- Can reconstruct from secret key and nonce at anytime
- Potential Support For Signing more than once (Stateful Signing since its n-1; can increase the size to allow more signing based on randomness of signatures)

#### Key Size

For a given digest message of 256-bits, we would have the following:

**SecretKey:** 32-64 bytes (can be reconstructed from this at any time)

**PublicKey:** Constant 16x keypairs hashed by variable length of `iter_hash = message_size_in_bits / 4` for full coverage from key_len (32-48).

**Signature:** 64x Keys of variable length for 256-bit length

**Verification-Time:** Much Longer But With Better Security Assumptions

**Signing:** Standard

#### Security Assumption

The Security is more advanced than that of typical winternitz because it requires longer verification and signing by using a map of 16x keys hashed to the `n` and only allowing `n-1` to appear. This means it can only have a certain amount of bytes labeled by the digest length.

#### Advanced Notes
- Nonce that helps secure the secret key from collisions by using keyed-hashing with an iterator. This design can be improved on.
- Only generating a `secret key` and `nonce` required
- Using BLAKE2
- 16 keys derived for 256-bit message (which can be derived from `secret key` and `nonce`)
- Provable Secure by hashing one less than the public key for each byte.
- Signature Size is quite small being that they only contain 16 keys.

## TODO:

- Look into more advanced key hashing methods.
- More advanced algorithm for iterating over the secret.
