# ZENYT Transaction Types

*Last updated: 2025-08-20*

## Overview

ZENYT implements a sophisticated transaction type system that balances efficiency for common operations with extensibility for complex use cases. All transactions operate on the funds-based ledger model using 64-bit colored coin amounts (60 bits amount + 4 bits color ID).

## Transaction Type Architecture

The type field uses an 8-bit encoding where:
- **Bits 0-6**: Transaction type family (0x00-0x7F)
- **Bit 7**: Amount distribution flag for types 0x01-0x03
  - **0**: Individual amounts (each participant has different amount)
  - **1**: Same amounts (uniform distribution, space-efficient encoding)

## Complete Transaction Type Catalog

### Type 0x00: Standard 1:1 Transfer

**Purpose**: Most efficient encoding for the most common case

**Data Structure**:
- Recipient address (20 bytes)
- Amount (64 bits, colored)

**Use Cases**:
- Basic payments between users
- Wallet-to-wallet transfers
- Simple commerce transactions

**Example**:
```rust
Transaction {
    header: TransactionHeader { /* common fields */ },
    data: TransactionData::Standard {
        recipient: Address::from_str("ZENYT-ABCDE-FGHJ5-KLMNP-QRSTU-VWXYZ-234567")?,
        amount: Amount::from_zenyt(1.5)?,
    }
}
```

### Type 0x01: M:N Transfer

**Purpose**: General many-to-many transfers for complex operations

**Data Structure**:
- Sender count (8 bits) - up to 256 senders
- Recipient count (8 bits) - up to 256 recipients  
- Senders array: Vec<(Address, Amount)>
- Recipients array: Vec<(Address, Amount)>

**Participant Limits**:
- **Maximum participants**: 512 (256 senders + 256 recipients)
- **Transaction level**: True M:N atomic operation
- **User level**: Typically 1:N (single user controlling multiple sender addresses)

**Use Cases**:
- **Exchange settlements**: Exchange hot wallets (M addresses) → user withdrawals (N addresses)
- **Pool distributions**: Mining pool reward addresses (M) → miners (N recipients)  
- **Multi-address consolidation**: User's scattered funds (M addresses) → recipients (N)
- **Complex group transactions**: Multi-party atomic settlements

**Bit 7 Variations**:
- **Bit 7 = 0**: Individual amounts for each participant
- **Bit 7 = 1**: Same amount for all recipients, drain-smallest-first for senders

**Example (Individual Amounts)**:
```rust
TransactionData::ManyToMany {
    senders: vec![
        (alice_addr, Amount::from_zenyt(2.0)?),
        (bob_addr, Amount::from_zenyt(3.0)?),
    ],
    recipients: vec![
        (charlie_addr, Amount::from_zenyt(1.5)?),
        (david_addr, Amount::from_zenyt(3.5)?),
    ],
}
```

**Example (Same Amount)**:
```rust
TransactionData::ManyToManySame {
    senders: vec![alice_addr, bob_addr, charlie_addr],
    recipients: vec![david_addr, eve_addr],
    amount_per_recipient: Amount::from_zenyt(1.0)?,
}
```

### Type 0x02: Genesis-to-Many (0:X)

**Purpose**: Protocol-driven distribution from genesis address

**Data Structure**:
- Recipient count as u16 (2 bytes, 0=1 recipient, max 65,536)
- Recipients array: Vec<(Address, Amount)>

**Participant Limits**:
- **Maximum participants**: 65,536 recipients
- **Genesis not counted**: Protocol action, not a participant
- **Rationale**: Designed for massive global distributions (carry-over to millions)
- **u16 vs u8**: 16-bit counters enable protocol-scale operations vs 8-bit for user transactions

**Special Properties**:
- No transaction fee (protocol authority)
- No signature verification required
- Genesis private key is public knowledge
- Validity determined by protocol consensus rules

**Use Cases**:
- **Carry-over distributions**: Global distribution to existing crypto holders
- **Mining rewards distribution**: Protocol rewards to miners
- **Demurrage redistribution**: Collected fees back to active addresses
- **Protocol-driven rewards**: System-level incentive distributions

**Example**:
```rust
Transaction::genesis_to_many(
    vec![miner1_addr, miner2_addr, miner3_addr],
    GenesisAmounts::Individual(vec![
        Amount::from_zenyt(5.0)?,
        Amount::from_zenyt(3.0)?, 
        Amount::from_zenyt(2.0)?,
    ]),
    timestamp,
)?
```

### Type 0x03: Many-to-Genesis (X:0)

**Purpose**: Collection to genesis address (dual-authority)

**Data Structure**:
- Sender count as u16 (2 bytes, 0=1 sender, max 65,536)
- Senders array: Vec<(Address, Amount)>

**Participant Limits**:
- **Maximum participants**: 65,536 senders
- **Genesis not counted**: Protocol destination, not a participant
- **Rationale**: Protocol needs to collect from many addresses simultaneously
- **u16 counters**: Enable mass collection operations (demurrage, dust cleanup)

**Authority Models**:
- **User-initiated**: Voluntary proof-of-burn, "pay to all" (requires signatures + fees)
- **Protocol-driven**: Demurrage collection (no fees, consensus-validated)

**Use Cases**:
- **Mass demurrage collection**: Automated collection from inactive addresses
- **Voluntary proof-of-burn**: Users burning funds for community benefit
- **"Pay to all" contributions**: Community donations via genesis
- **Dust address cleanup**: Protocol collection of negligible balances

**Example (User-Initiated)**:
```rust
TransactionData::ManyToGenesis {
    senders: vec![
        (user_addr, Amount::from_zenyt(10.0)?),
    ],
    // Requires user signature and fee payment
}
```

### Type 0x04: Message/Data Storage

**Purpose**: Store arbitrary data on the ledger

**Data Structure**:
- Message length (16 bits)
- Message payload (variable, up to 64KB)

**Use Cases**:
- Timestamping important documents
- Public announcements
- Small data archival
- Notarization services

**Example**:
```rust
TransactionData::Message {
    data: b"Proof of existence for document ABC123 at 2025-08-20".to_vec(),
}
```

### Type 0x05: Smart Contract Operation

**Purpose**: Contract deployment and execution

**Data Structure**:
- Contract address (20 bytes)
- Method call identifier (32 bits)
- Parameters (variable length)

**Use Cases**:
- Contract deployment
- Contract method execution
- Automated transactions
- Complex business logic

**Example**:
```rust
TransactionData::SmartContract {
    contract_address: contract_addr,
    method: "transfer".to_string(),
    parameters: vec![param1, param2, param3],
}
```

### Type 0x06: Tethered Asset Operation

**Purpose**: Real-world asset linkage and metadata management

**Data Structure**:
- Asset hash (32 bytes) - content-addressed identifier
- Operation type (8 bits) - create, update, transfer, etc.
- Metadata size (16 bits) - up to 64KB
- Metadata blob (variable) - JSON, CBOR, or other structured data

**Use Cases**:
- Property deeds and titles
- Academic certificates
- Legal contracts
- Supply chain tracking
- Asset digitization

**Example**:
```rust
TransactionData::TetheredAsset {
    asset_hash: [0x12, 0x34, /* ... 32 bytes ... */],
    operation: TetheredAssetOp::Create,
    metadata: serde_json::to_vec(&PropertyDeed {
        address: "123 Main St, Anytown, USA",
        owner: "John Smith",
        legal_description: "Lot 1, Block 2, Subdivision ABC",
        // ... additional fields
    })?,
}
```

### Type 0xFF: Protocol Extensions

**Purpose**: Reserved for future protocol upgrades

**Data Structure**: Extensible format for future needs

**Use Cases**:
- Protocol version upgrades
- New transaction features
- Backward compatibility maintenance
- Emergency protocol adjustments

## Participant Limits Summary

Understanding participant limits across transaction types:

| Transaction Type | Max Participants | Counter Type | Rationale |
|------------------|------------------|--------------|-----------|
| **Type 0x01: M:N** | 512 (256+256) | u8 + u8 | User transactions, practical limits |
| **Type 0x02: Genesis→Many** | 65,536 | u16 | Protocol-scale distributions |
| **Type 0x03: Many→Genesis** | 65,536 | u16 | Mass collection operations |

**Key Insights**:
- **Genesis is not a participant**: Protocol action, not counted in limits
- **M:N dual perspective**: Transaction-level (true M:N) vs user-level (1:N pattern)
- **Counter sizing rationale**: u8 for users (practical), u16 for protocol (massive scale)

## Same-Amount Distribution Logic (Bit 7 = 1)

For transaction types 0x01, 0x02, and 0x03, the "same amount" flag provides significant advantages:

### Encoding Efficiency
- **Individual amounts**: Store participant count + array of amounts
- **Same amounts**: Store participant count + single amount value
- **Space savings**: Dramatic reduction for large participant counts

### M:N Same-Amount Semantics (Type 0x01 + bit 7)
- **Recipient perspective**: Each recipient receives identical amount
- **Sender draining**: "Drain smallest first" strategy
  1. Start with lowest balance senders
  2. Empty them completely before moving to larger senders
  3. Preserves maximum value during dust rescue operations
- **User control**: Single user typically controls M sender addresses

### Dust Rescue Operations

The drain-smallest-first strategy serves crucial value preservation:

**Problem**: Dust collector automatically reclaims negligible balances
**Solution**: Pool operators include dust addresses as senders in M:N transactions
**Result**: Dust funds rescued for productive use instead of system collection
**Benefit**: Preserves value while enabling efficient uniform distributions

## Validation Rules

All transaction types must satisfy comprehensive validation:

### Universal Requirements
- **Balance conservation**: Total inputs = total outputs + fees
- **Color consistency**: Cannot mix colors within single amount field
- **Demurrage application**: Balances adjusted for time elapsed since last activity
- **Address format**: All addresses must pass Reed-Solomon validation

### Authority-Specific Requirements
- **User transactions**: Valid Ed25519 signatures required
- **Protocol transactions**: Consensus rules determine validity
- **Genesis transactions**: No fees, no signatures (protocol authority)

### Type-Specific Limits
- **Participant counts**: Respect maximum limits for each transaction type
- **Data sizes**: Message and metadata payloads within size limits
- **Color compatibility**: Colored coin rules enforced

## Implementation Notes

### Implementation
The complete transaction system is implemented in Jantar v0.2.0 with full support for all 8 transaction types, comprehensive validation, and extensive test coverage.

### Key Data Structures
```rust
pub enum TransactionType {
    Standard = 0x00,
    ManyToMany = 0x01,
    GenesisToMany = 0x02,
    ManyToGenesis = 0x03,
    Message = 0x04,
    SmartContract = 0x05,
    TetheredAsset = 0x06,
    ProtocolExtension = 0xFF,
}

pub struct Transaction {
    pub header: TransactionHeader,
    pub data: TransactionData,
}

pub struct TransactionHeader {
    pub version: u8,
    pub tx_type: TransactionType,
    pub timestamp: u32,
    pub fee: Amount,
    pub signature: Signature,
    pub public_key: VerifyingKey,
}
```

### Testing and Validation
All transaction types include comprehensive test coverage:
- Unit tests for each transaction type
- Integration tests for complex scenarios
- Validation boundary condition testing
- Performance benchmarks for large transactions

## Future Extensions

The transaction type system is designed for extensibility:

- **Type 0xFF**: Provides upgrade mechanism for new features
- **Versioning**: Transaction version field allows incremental improvements
- **Metadata flexibility**: Tethered assets support arbitrary structured data
- **Smart contracts**: Execution environment for complex logic

This architecture ensures ZENYT can evolve while maintaining backward compatibility and providing sophisticated transaction capabilities from day one.