# Dual-VM Execution Model
description: "How Paxeer routes transactions across EVM and ArgusVM for optimal execution"

:::scalar-callout{type="info"}
Audience: protocol engineers extending execution, router logic, or precompile bridges.
:::

## 1. Execution Router
- Inspects transaction target:
  - **EVM bytecode / Solidity contracts** → EVM path.
  - **AVM/OpenLang artifacts** → AVM path.
  - **Precompile addresses** (`0x100+`) → bridge calls into AVM-native services.
- Preserves **unified state root** and Ethereum-shaped receipts/logs for tooling compatibility.

### Pseudocode Sketch
```go
func Execute(tx Tx) (Receipt, error) {
  switch classify(tx.To, tx.Payload) {
  case EVM:
    return evm.Exec(tx)
  case AVM:
    return avm.Exec(tx)
  case PRECOMPILE:
    return bridge.Exec(tx)
  }
}
```

## 2. EVM Path
- Stack-based, 256-bit words; Ethereum gas schedule.
- Compatibility goal: Solidity/Vyper + existing toolchains.
- Uses standard precompiles (0x01–0x0F) plus Paxeer bridge precompiles.
- Receipts/logs: Ethereum format; required for explorer/tooling.

## 3. AVM Path (ArgusVM)
- Register-based (32 × 256-bit registers), fixed 64-bit instruction width.
- Memory segments: code, data, stack, heap; strict bounds.
- Gas model: fixed opcode costs; cold/warm storage; memory expansion formula.
- Syscall table: crypto (ECDSA/BLS), hashing, modexp, BN256, etc.
- Precompiles: extended set for cryptography; exposed at EVM-facing addresses.
- Determinism rules: no time/entropy/I/O; division by zero returns 0; OOB memory reverts.

:::scalar-callout{type="warning"}
When adding opcodes or syscalls to AVM, keep deterministic guarantees: fixed gas, no host I/O, bounded memory.
:::

## 4. Precompile Bridge (Cross-VM)
- EVM-visible addresses mapped to AVM-native logic:
  - `0x100` Capital
  - `0x101` VMBridge
  - `0x102` RiskEngine
- Pattern: EVM call → bridge shim → AVM syscall/op → returns to EVM with ABI-compatible output.
- Ensure **gas parity**: charge EVM-side for bridge costs; enforce AVM gas internally.

## 5. State Commit & Receipts
- Both VMs mutate the same state backend (accounts/storage tries).
- Block import collates receipts/logs; bloom filters built as in Ethereum.
- Consensus seals the state root; light clients verify via Merkle proofs.

## 6. Testing Guidance
- Golden tests: identical tx replay across nodes must yield identical receipts/state root.
- Differential tests: EVM vs AVM bridging correctness for shared precompiles.
- Gas accounting: assert invariants on gas used per opcode batch.

## 7. Extension Hooks
- Add new AVM syscalls → extend syscall table + cost map.
- Add new precompile → register bridge address + ABI + gas schedule.
- Router policy tweaks → adjust classifier; consider tx metadata or code markers.
