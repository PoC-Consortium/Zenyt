# Ethereum Address Generation - Complete Implementation Guide

## Overview
Generate an Ethereum address from a private key using secp256k1 elliptic curve cryptography and Keccak-256 hashing.

## Step-by-Step Process

### 1. Private Key
- **Format**: 32-byte (256-bit) random number
- **Range**: 1 to (secp256k1 curve order - 1)
- **Encoding**: Big-endian bytes
- **Example**: `0x4c0883a69102937d6231471b5dbb6204fe5129617082792ae468d01a3f362318`

### 2. Generate Public Key
- **Curve**: secp256k1 (same as Bitcoin)
- **Method**: Elliptic curve point multiplication: `pubkey = privkey * G`
- **Output**: Uncompressed format (65 bytes total)
- **Structure**: `0x04 || X-coordinate (32 bytes) || Y-coordinate (32 bytes)`
- **For address derivation**: **Remove the 0x04 prefix byte**
- **Working data**: 64 bytes (X + Y coordinates only)

### 3. Hash Public Key
- **Algorithm**: Keccak-256 (NOT SHA-3, they're different!)
- **Input**: 64-byte uncompressed public key (without 0x04 prefix)
- **Output**: 32-byte hash

### 4. Extract Address
- **Method**: Take the **last 20 bytes** of the Keccak-256 hash
- **Result**: 20-byte Ethereum address

### 5. Apply EIP-55 Checksum
```
Input: 20-byte address as lowercase hex string (no 0x prefix)
Process:
    1. hash_for_checksum = Keccak256(lowercase_address_string)
    2. For each character at position i in lowercase_address_string:
        - If character is a digit (0-9): keep unchanged
        - If character is a letter (a-f):
            - If hash_for_checksum[i] >= 0x8: make uppercase
            - Else: keep lowercase
    3. Prepend "0x" to result
```

## Rust Implementation Notes

**Required Dependencies:**
- `secp256k1` crate for elliptic curve operations
- `tiny-keccak` or `sha3` crate for Keccak-256 hashing
- `hex` crate for hexadecimal encoding/decoding

**Key Functions Needed:**
```rust
fn private_key_to_public_key(privkey: &[u8; 32]) -> [u8; 64]
fn keccak256(input: &[u8]) -> [u8; 32] 
fn apply_eip55_checksum(address: &str) -> String
```

**Critical Details:**
- Use uncompressed public key format (remove 0x04 prefix before hashing)
- Keccak-256 is different from SHA-3 standard
- EIP-55 checksum operates on hex string characters, not bytes
- Final address format: `0x` + 40 hex characters with mixed case

This should provide the exact algorithmic precision needed for a correct Rust implementation.
