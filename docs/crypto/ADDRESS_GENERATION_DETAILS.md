# Address Generation Implementation Details

*Last updated: 2025-01-08*

## Overview

This document provides technical details on how each cryptocurrency family generates addresses in Jantar, including the complete cryptographic pipeline from entropy to final address format.

## 1. ZENYT Address Generation

### Pipeline
```
Entropy → Ed25519 Private Key → Ed25519 Public Key → Blake2b-160 → Reed-Solomon → ZENYT-XXXXX-XXXXX-XXXXX-XXXXX-XXXXXX
```

### Implementation Details

**File:** `src/crypto/families/zenyt.rs`

1. **Key Generation**
   ```rust
   let mut csprng = OsRng;
   let signing_key = SigningKey::generate(&mut csprng);
   ```
   - Uses Ed25519-dalek library
   - 32-byte (256-bit) private key
   - Deterministic public key derivation

2. **Address Derivation**
   ```rust
   fn pubkey_to_address(&self, pubkey: &VerifyingKey) -> [u8; 20] {
       let mut hasher = Blake2b::<U20>::new();
       hasher.update(pubkey.as_bytes());
       let hash = hasher.finalize();
       // Returns 20-byte address
   }
   ```
   - Blake2b hash truncated to 160 bits (20 bytes)
   - More modern than SHA256/RIPEMD160 used by Bitcoin

3. **Reed-Solomon Encoding**
   ```rust
   fn address_to_rs_string(&self, address: &[u8; 20]) -> Result<String>
   ```
   - Converts 20 bytes → Base32 digits (0-31)
   - Adds 4 check symbols for error correction
   - Total: 31 symbols (27 data + 4 check)
   - Alphabet: `23456789ABCDEFGHJKLMNPQRSTUVWXYZ` (no 0,1,I,O)

4. **Final Format**
   - Pattern: `ZENYT-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX-XXXXXX`
   - 5 groups of 5 characters + 1 group of 6 characters
   - Error detection and correction capability

### Example
```
Private Key: 0x1234...abcd (32 bytes)
Public Key:  0xfedc...4321 (32 bytes)
Blake2b-160: 0xa1b2...ef01 (20 bytes)
RS Encoded:  ZENYT-ABCDE-FGHJ5-KLMNP-QRSTU-VWXYZ-234567
```

## 2. Bitcoin Family (BTC, LTC, DOGE)

### Pipeline
```
Entropy → secp256k1 Private Key → secp256k1 Public Key → SHA256 → RIPEMD160 → Base58Check → Address
```

### Implementation Details

**File:** `src/crypto/families/bitcoin.rs`

1. **Key Generation**
   ```rust
   let mut rng = OsRng;
   let (secret_key, public_key) = self.secp.generate_keypair(&mut rng);
   ```
   - secp256k1 elliptic curve
   - 32-byte private key
   - 33-byte compressed public key

2. **Address Derivation**
   ```rust
   // SHA256 + RIPEMD160 = Bitcoin's Hash160
   let sha = sha256::Hash::hash(&public_key_bytes);
   let hash160 = ripemd160::Hash::hash(&sha);
   ```

3. **Network-Specific Prefixes**
   ```rust
   struct NetworkParams {
       p2pkh_prefix: u8,    // Pay-to-PubKey-Hash
       p2sh_prefix: u8,     // Pay-to-Script-Hash
       wif_prefix: u8,      // Wallet Import Format
   }
   ```
   
   | Coin | P2PKH Prefix | P2SH Prefix | WIF Prefix | Address Starts With |
   |------|-------------|------------|------------|-------------------|
   | BTC  | 0x00        | 0x05       | 0x80       | '1' or '3'        |
   | LTC  | 0x30        | 0x32       | 0xB0       | 'L' or 'M'        |
   | DOGE | 0x1E        | 0x16       | 0x9E       | 'D'               |

4. **Base58Check Encoding**
   ```rust
   let mut payload = vec![params.p2pkh_prefix];
   payload.extend_from_slice(&hash160);
   let checksum = &sha256d::Hash::hash(&payload)[..4];
   payload.extend_from_slice(checksum);
   bs58::encode(payload).into_string()
   ```
   - Double SHA256 for checksum
   - 4-byte checksum appended
   - Base58 alphabet (no 0, O, I, l)

### Example (Bitcoin)
```
Private Key:  0x1234...abcd (32 bytes)
Public Key:   0x02abcd...ef12 (33 bytes compressed)
SHA256:       0x5678...9abc (32 bytes)
RIPEMD160:    0xdef0...1234 (20 bytes)
With Prefix:  0x00def0...1234 (21 bytes)
Checksum:     0xabcd (4 bytes)
Base58:       1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa
```

## 3. Ethereum

### Pipeline
```
Entropy → secp256k1 Private Key → Uncompressed Public Key → Keccak256 → Last 20 bytes → 0x prefix
```

### Implementation Details

**File:** `src/crypto/families/ethereum.rs`

1. **Key Generation**
   ```rust
   let mut rng = OsRng;
   let secret_key = SecretKey::new(&mut rng);
   let public_key = PublicKey::from_secret_key(&self.secp, &secret_key);
   ```
   - Same curve as Bitcoin (secp256k1)
   - But uses uncompressed public keys (65 bytes)

2. **Address Derivation**
   ```rust
   let uncompressed = &public_key.serialize_uncompressed()[1..]; // Skip 0x04 prefix
   let mut hasher = Keccak::v256();
   hasher.update(uncompressed);
   let mut hash = [0u8; 32];
   hasher.finalize(&mut hash);
   // Take last 20 bytes
   let address_bytes = &hash[12..];
   ```
   - Keccak256 (not SHA3-256!)
   - Takes last 160 bits (20 bytes)
   - No additional encoding

3. **Checksum (EIP-55)**
   ```rust
   fn to_checksum_address(&self, address: &str) -> String {
       let address_lower = address[2..].to_lowercase();
       let hash = keccak256(address_lower.as_bytes());
       // Capitalize based on hash bits
   }
   ```
   - Mixed case for error detection
   - Each hex character checked against hash

### Example
```
Private Key:     0x1234...abcd (32 bytes)
Public Key:      0x04abcd...ef12 (65 bytes uncompressed)
Without prefix:  0xabcd...ef12 (64 bytes)
Keccak256:       0x5678...9abc (32 bytes)
Last 20 bytes:   0xdef0...1234
Final Address:   0xdEf0123456789aBcDeF0123456789aBcDeF01234
```

## 4. NXT Family (BURST, NXT, ARDOR)

### Pipeline
```
Passphrase → SHA256 → Curve25519 Private Key → X25519 Public Key → SHA256 → First 8 bytes → Reed-Solomon
```

### Implementation Details

**File:** `src/crypto/families/nxt.rs`

1. **Passphrase-Based Generation**
   ```rust
   fn generate_keypair(&self) -> Result<KeyPair> {
       let mut rng = rand::thread_rng();
       let passphrase = generate_random_passphrase(&mut rng);
       self.keypair_from_passphrase(&passphrase)
   }
   ```
   - Generates random passphrase from word list
   - Alternative: user provides passphrase

2. **Key Derivation**
   ```rust
   let mut hasher = Sha256::new();
   hasher.update(passphrase.as_bytes());
   let hash = hasher.finalize();
   // Clamp for Curve25519
   secret_bytes[0] &= 0xf8;
   secret_bytes[31] &= 0x7f;
   secret_bytes[31] |= 0x40;
   ```
   - SHA256 of passphrase → private key
   - Curve25519 scalar clamping

3. **Account ID Generation**
   ```rust
   let mut hasher = Sha256::new();
   hasher.update(&public_key_bytes);
   let hash = hasher.finalize();
   // Take first 8 bytes as little-endian u64
   let account_id = u64::from_le_bytes(first_8_bytes);
   ```

4. **Reed-Solomon Encoding**
   ```rust
   fn encode_account_id(account_id: u64, prefix: &str) -> String
   ```
   - Different from ZENYT's RS encoding
   - 17 symbols: 13 data + 4 check
   - Format: `BURST-XXXX-XXXX-XXXX-XXXXX`

### Example (BURST)
```
Passphrase:    "correct horse battery staple"
SHA256:        0x1234...abcd (32 bytes)
Private Key:   0x1234...abc0 (32 bytes, clamped)
Public Key:    0xfedc...4321 (32 bytes)
SHA256:        0x8765...4321 (32 bytes)
Account ID:    12345678901234567890 (u64)
RS Address:    BURST-ABCD-EFGH-JKLM-NOPQR
```

## Key Differences Summary

| Aspect | ZENYT | Bitcoin | Ethereum | NXT/BURST |
|--------|-------|---------|----------|-----------|
| **Curve** | Ed25519 | secp256k1 | secp256k1 | Curve25519 |
| **Hash** | Blake2b-160 | SHA256+RIPEMD160 | Keccak256 | SHA256 |
| **Encoding** | Reed-Solomon | Base58Check | Hex | Reed-Solomon |
| **Checksum** | 4 RS symbols | 4 bytes | EIP-55 | 4 RS symbols |
| **Entropy** | OsRng | OsRng | OsRng | Passphrase |
| **Key Size** | 32 bytes | 32 bytes | 32 bytes | 32 bytes |
| **Address Length** | 37 chars | 26-35 chars | 42 chars | 21 chars |

## Security Considerations

1. **ZENYT**: Modern crypto (Ed25519 + Blake2b), error correction built-in
2. **Bitcoin**: Battle-tested, double-SHA256 checksums, widespread adoption
3. **Ethereum**: Simple design, case-checksum for typo detection
4. **NXT/BURST**: Passphrase-based can be weak if passphrase is poor

## Testing Address Generation

```bash
# Generate and verify each type
./target/release/jantar << EOF
generate BTC
generate ETH
generate BURST
generate ZENYT
quit
EOF

# Verify addresses are valid
./target/release/jantar << EOF
validate 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa
validate 0x742d35Cc6634C0532925a3b844Bc7e1292A5E223
validate BURST-ABCD-EFGH-JKLM-NOPQR
validate ZENYT-ABCDE-FGHJ5-KLMNP-QRSTU-VWXYZ-234567
quit
EOF
```

## References

- [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki): HD Wallets
- [EIP-55](https://eips.ethereum.org/EIPS/eip-55): Ethereum Address Checksum
- [Ed25519](https://ed25519.cr.yp.to/): High-speed signatures
- [Reed-Solomon Codes](https://en.wikipedia.org/wiki/Reed%E2%80%93Solomon_error_correction)