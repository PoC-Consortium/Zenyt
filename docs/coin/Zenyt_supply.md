
Envisioned for Zenyt is a total capped supply of 11 bn, where the
divisibility is same as with BTC (100 mio); The smallest unit is
called Planck (like Satoshi is for BTC). THis means Zenyt total supply
is 1100000000000000000 Planck This nicely fits in 60bit 2^60-1 =
1152921504606846975 and is roughly 1/16 of what 64bit can offer. The
remaining 4 bit (most significant should be used for this - I think -
but maybe it does not matter with the right data structure) could be
used for having up to 16 "colored coins".

## Implementation Status (v0.2.0)

The complete ZENYT transaction system is now implemented in Jantar v0.2.0, providing:

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

## Revolutionary Transaction System

The ZENYT transaction system implements 8 sophisticated transaction types:

### Transaction Types Overview

1. **Type 0x00: Standard 1:1 Transfer**
   - Most efficient encoding for common payments
   - Single sender to single recipient
   - Optimized for wallet-to-wallet transfers

2. **Type 0x01: M:N Transfer** 
   - Many-to-many atomic transactions supporting up to 512 participants
   - Transaction level: True M:N (256 senders + 256 recipients)
   - User level: Typically 1:N (single user controlling multiple addresses)
   - Supports exchange settlements, pool distributions, and multi-address consolidation
   - Includes "same-amount" flag for uniform distributions with drain-smallest-first

3. **Type 0x02: Genesis-to-Many (0:X)**
   - Protocol-driven distribution supporting up to 65,536 participants  
   - Genesis not counted (protocol action, not participant)
   - Designed for massive global distributions (carry-over to millions)
   - u16 counters enable protocol-scale operations vs u8 for user transactions
   - No transaction fees (protocol authority)

4. **Type 0x03: Many-to-Genesis (X:0)**
   - Collection supporting up to 65,536 participants
   - Genesis not counted (protocol destination, not participant)  
   - Enables mass demurrage collection and dust cleanup
   - u16 counters for large-scale protocol operations
   - Dual authority: user-initiated (with fees) or protocol-driven (consensus)

5. **Type 0x04: Message/Data Storage**
   - Store arbitrary data on the ledger
   - Timestamping and announcements
   - Small data archival capabilities

6. **Type 0x05: Smart Contract Operation**
   - Contract deployment and execution
   - Automated transactions with complex business logic

7. **Type 0x06: Tethered Asset Operation**
   - Real-world asset linkage and metadata management
   - Property deeds, certificates, legal documents
   - Up to 64KB metadata per transaction

8. **Type 0xFF: Protocol Extensions**
   - Reserved for future protocol upgrades
   - Ensures forward compatibility

### Genesis-Based Architecture

Unlike traditional cryptocurrencies where rewards appear "out of thin air", ZENYT creates the entire 11 billion supply at genesis and stores it in a special genesis address:

- All mining rewards are transactions from genesis to miners
- Demurrage collection flows back to genesis
- "Payment to all" transactions supported via genesis
- System invariant: Genesis + Ledger = Total Supply
- No mining endpoint - demurrage continuously replenishes reward pool

### Same-Amount Distribution Patterns

The "same amount" flag (bit 7) provides space-efficient encoding for uniform distributions:

- **Drain-smallest-first strategy**: For dust rescue operations
- **Space savings**: Single amount value instead of per-participant amounts
- **Value preservation**: Rescues dust addresses from dust collector

### Validation Framework

All transactions must satisfy:
- Balance conservation (inputs = outputs + fees)
- Color consistency (cannot mix colors within single amount field)
- Demurrage application (time-based balance adjustments)
- Signature verification (for user transactions)
- Address format validation (Reed-Solomon)
- Type-specific limits (participant counts, metadata sizes)
