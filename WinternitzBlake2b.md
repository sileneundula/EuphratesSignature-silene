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

## 2. Naive Construction of WOTSVARLEN

This is a naive construction of WOTSVARLEN using BLAKE2.

### Assumptions

We assume we are signing a message of 256-bits (32-bytes).

We assume we are using the `w` parameter of `16`, signing 4 bits at a time (0-15).

We assume we are using BLAKE2B with variable digest length from 32-48.

### Worst Method: Mapping Method For Short Secret Key Length

1. We generate 4 secret keys and one `nonce`
2. We take sk = BLAKE2B(var_digest,key) for a total of 64 keys each being 32-48 bytes in length from the same input. Each key is labeled by its length.
3. Each key is hashed by BLAKE2B(32) as the public key. This means there are 64 public keys.
4. To sign a message, we take the given length of the 4 bits (0-15) and hash it for the digest length using a key for each that is derived from the `nonce xor position(i)`.
5. Our signature is then, `sk`

### Construction

1. We generate 64 secret keys for a 256-bit message from randomness and hash them once using `BLAKE2B(32)`. This is our `sk`.
2. We generate the public key using 1x each `sk`
3. We sign a message by using a hash digest of {32-48} for the number of bits to be signed.

### Mapping Technique

1. We generate 64 secret keys for a 256-bit message
2. We map out the keys, creating a BLAKE2B(
