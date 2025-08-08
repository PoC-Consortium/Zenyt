# Self-Update Mechanism

The Jantar P2P application now includes a self-update mechanism with ed25519 signature verification.

## How it Works

1. **Version Announcements**: When peers connect, they automatically announce their version to the network via gossipsub.

2. **Update Discovery**: 
   - Use the `update` command to check if any peer has a newer version
   - The system compares versions and identifies the newest available

3. **Update Distribution**:
   - When a peer requests an update, the node with the newer version sends it
   - Updates are distributed as signed binaries via the gossipsub network

4. **Signature Verification**:
   - All updates must be signed with a valid ed25519 signature
   - The signature is verified before applying the update
   - Only updates with valid signatures are installed

## Commands

- `version` - Show current version
- `update` - Check for and install updates from peers

## Development: Signing Updates

To create a signed update for testing:

```bash
# Build the new version
cargo build --release

# Sign the binary
cargo run --bin sign-update -- \
  --binary target/release/jantar \
  --version 0.2.0 \
  --key signing_key.hex  # Optional, uses test key if not provided
```

This creates:
- A signature for the binary
- An update info file with version, size, and SHA256 hash

## Security

- Updates are verified using ed25519 signatures
- The public key is embedded in the application
- Only binaries signed with the corresponding private key can be installed
- The current binary is backed up before applying updates

## Version Format

Versions follow semantic versioning: `MAJOR.MINOR.PATCH`
- Updates are only applied if the new version is greater than the current version
- Version comparison follows standard semver rules