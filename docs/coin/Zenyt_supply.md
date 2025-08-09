Envisioned for Zenyt is a total capped supply of 11 bn, where the
divisibility is same as with BTC (100 mio); The smallest unit is
called Planck (like Satoshi is for BTC). THis means Zenyt total supply
is 1100000000000000000 Planck This nicely fits in 60bit 2^60-1 =
1152921504606846975 and is roughly 1/16 of what 64bit can offer. The
remaining 4 bit (most significant should be used for this - I think -
but maybe it does not matter with the right data structure) could be
used for having up to 16 "colored coins".

An initial implementation is at src/coin/amount.rs

This implementation provides:

Key Features:

60-bit amounts with 4-bit color IDs packed into a single u64
11 billion ZENYT cap with overflow protection
16 colored coins support (Color IDs 0-15)
Arithmetic operations with color compatibility checking
Demurrage support for your decay calculations
Serde support for database storage
Display/parsing for human-readable amounts
Design Decisions:

Color ID in upper 4 bits (easier bit manipulation)
60-bit amount limit enforced for all colors
Additional 11B limit for standard ZENYT (Color 0)
Result-based arithmetic to handle overflows gracefully
Planck as base unit (like Satoshi) for precision
Usage Examples:

￼
rust
// Standard ZENYT
let zenyt = Amount::from_zenyt(1.5)?; // 1.5 ZENYT
let planck = Amount::new(150_000_000, ColorId::ZENYT)?;

// Colored coins  
let token = Amount::new(1000, ColorId::new(3)?)?;

// Arithmetic (same color only)
let sum = (zenyt + planck)?;

// Demurrage
let after_decay = amount.apply_demurrage(1.0)?; // 1% reduction
